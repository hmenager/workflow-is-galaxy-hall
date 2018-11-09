# *cmsearch-multimodel-wf* workflow test with Galaxy-CWL

## Overview

cmsearch-multimodel-wf is a CWL Workflow part of ELIXIR project.

In order to run cmsearch-multimodel-wf workflow with Galaxy-CWL, some
modifications must be made in software and CWL files below

* Galaxy-CWL 
* Cwltool 
* CWL Tools files
* CWL Workflow files

This document describes those modifications.

## Repositories used

Galaxy-CWL

```
https://github.com/hmenager/galaxy
```

Cwltool

```
https://github.com/hmenager/cwltool
```

ELIXIR Workflow

```
https://github.com/hmenager/workflow-is-cwl
```

## Modifications overview

Main modifications are:

* Prevent EDAM filetype checking in Cwltool
* Add 'Directory' type support in Galaxy (using binding between tar-file and directory-type)
* Add 'gx:interface' hints in CWL tools files
* Replace relative-path with absolute-path in 'run' attributes of CWL workflow files (tools links).
* Call '_init_dynamic_tools' method at Galaxy startup to populate '_tools_by_hash' variable.
* Assume yaml CWL workflow if exception occurs during json deserialization (when user click on 'import')

## Modifications description

This section contains a merge of issues from this repository

```
https://github.com/hmenager/workflow-is-cwl/issues
```

Each workflow and tools have a dedicated issue (most are now closed, but can be reopen if new problems are found).

### Galaxy-CWL

#### Enable CWL workflow execution with GUI

'_tools_by_hash' must be initialized at galaxy startup to prevent error below
(which occurs when running a CWL workflow):

```
Traceback (most recent call last):
  File "lib/galaxy/web/framework/decorators.py", line 285, in decorator
    rval = func(self, trans, *args, **kwargs)
  File "lib/galaxy/webapps/galaxy/api/workflows.py", line 417, in workflow_dict
    ret_dict = self.workflow_contents_manager.workflow_to_dict(trans, stored_workflow, style=style)
  File "lib/galaxy/managers/workflows.py", line 384, in workflow_to_dict
    return self._workflow_to_dict_run(trans, stored)
  File "lib/galaxy/managers/workflows.py", line 408, in _workflow_to_dict_run
    module_injector.inject(step, steps=workflow.steps, exact_tools=False)
  File "lib/galaxy/workflow/modules.py", line 1505, in inject
    module = step.module = module_factory.from_workflow_step(self.trans, step, **kwargs)
  File "lib/galaxy/workflow/modules.py", line 1382, in from_workflow_step
    return self.module_types[type].from_workflow_step(trans, step, **kwargs)
  File "lib/galaxy/workflow/modules.py", line 815, in from_workflow_step
    module = super(ToolModule, Class).from_workflow_step(trans, step, tool_id=tool_id, tool_version=tool_version, tool_hash=tool_hash, **kwds)
  File "lib/galaxy/workflow/modules.py", line 88, in from_workflow_step
    module = Class(trans, **kwds)
  File "lib/galaxy/workflow/modules.py", line 772, in __init__
    self.tool = trans.app.toolbox.get_tool(tool_id, tool_version=tool_version, exact=exact_tools, tool_hash=tool_hash)
  File "lib/galaxy/tools/toolbox/base.py", line 439, in get_tool
    tool_id = self._tools_by_hash[tool_hash].id
KeyError: u'e4de79296ec91ab9b8d8d9d71f94044a2561c01b9fc708e5197d432b453fa297'
```

'filter' has been replaced by 'filter_by' to prevent error below:

```
  File "lib/galaxy/workflow/modules.py", line 771, in __init__
    trans.app.toolbox._init_dynamic_tools()
  File "lib/galaxy/tools/toolbox/base.py", line 183, in _init_dynamic_tools
    for dynamic_tool in self.app.dynamic_tool_manager.list_tools():
  File "lib/galaxy/managers/tools.py", line 117, in list_tools
    return self.query().filter(active=active)
TypeError: filter() got an unexpected keyword argument 'active'
```

9e0d85b  

#### Enable CWL workflow import with GUI

To prevent exception below

```
raise exceptions.MessageException("The data content does not appear to be a valid workflow.")
```

used the hack below

```
  def __api_import_from_archive(self, trans, archive_data, source=None, from_path=None):
      try:
          data = json.loads(archive_data)
      except ValueError:

          # hack
          # if not json, assume the file to be a cwl workflow
          # FIXME: don't use fmt to determine if cwl or galaxy wf, as galaxy wf use yaml fmt as well (galaxy-wf-fmt-2)
          if from_path is not None:
              print('{}'.format("CWL-IS: assume CWL workflow"))
              data = {"src": "from_path", "path": from_path}
          else:
              raise exceptions.MessageException("The data content does not appear to be a valid workflow.")
  
      ...
```

f2a4645

#### Rename "Test Dataset".

To prevent error below when running CWL workflow 'test_simplest_wf' test

```
  File "/home/jra001k/snapshot/pasteur/galaxy/.venv/local/lib/python2.7/site-packages/cwltool/pathmapper.py", line 46, in visit_class
    op(rec)
  File "/home/jra001k/snapshot/pasteur/galaxy/.venv/local/lib/python2.7/site-packages/cwltool/command_line_tool.py", line 186, in check_adjust
    raise WorkflowException("Invalid filename: '%s' contains illegal characters" % (f["basename"]))
WorkflowException: Invalid filename: 'Test Dataset' contains illegal characters
```

replaced space with underscore ('Test Dataset' to 'Test_Dataset') in test/base/populators.py

```
def upload_payload(self, history_id, content=None, **kwds):
    name = kwds.get("name", "Test_Dataset")

    ...
```

7827974

#### Prevent flooding Galaxy left panel with tools description and label.

```
class CommandLineToolProxy(ToolProxy):
    _class = "CommandLineTool"

    def description(self):
        return ''
        #return self._tool.tool.get('doc')

    def label(self):
        label = self._tool.tool.get('label')

        if label is not None:
            return label.partition(":")[0] # return substring before ':'
        else:
            return ''
```

ad2f92b  

#### Map tar file to 'Directory' type.

```

for k, v in input_json.iteritems():
    if isinstance(v, dict) and v['class'] == 'File' and v['nameext'] == '.tar':
        print("CWL-IS: tar files uploaded in Galaxy are interpreted as 'Directory'.")

        tar_file_location = v['location']
        directory_name = v['nameroot']

        tmp_dir = os.path.join('/tmp', str(uuid.uuid4()))
        directory_location = os.path.join(tmp_dir, directory_name)

        os.makedirs(tmp_dir)

        bkp_cwd = os.getcwd(); os.chdir(tmp_dir)
        tar = tarfile.open(tar_file_location); tar.extractall(); tar.close()
        os.chdir(bkp_cwd)


        v['class'] = 'Directory'
        v['location'] = directory_location
        v['nameext'] = 'None'
        v['nameroot'] = 'example'
        v['basename'] = 'example'
        #v['size'] = 
```

22cc09f  

Related code (not used for now)

```
convert_response = self.dataset_populator.run_tool(
    tool_id="CONVERTER_tar_to_directory",
    inputs={"input1": {"src": "hda", "id": create_response.json()["outputs"][0]["id"]}},
    history_id=history_id,
)
```

```
lib/galaxy/datatypes/converters/tar_to_directory.xml

<tool id="CONVERTER_tar_to_directory" name="Convert tar to directory" version="1.0.0" profile="17.05">
    <command>
        mkdir '$output1.files_path';
        cd '$output1.files_path';
        tar -xzf '$input1'
    </command>
    <inputs>
        <param format="tar" name="input1" type="data"/>
    </inputs>
    <outputs>
        <data format="directory" name="output1"/>
    </outputs>
    <help>
    </help>
</tool>
```

```
parameter_types = dict(
    text=TextToolParameter,
    integer=IntegerToolParameter,
    float=FloatToolParameter,
    boolean=BooleanToolParameter,
    genomebuild=GenomeBuildParameter,
    select=SelectToolParameter,
    color=ColorToolParameter,
    data_column=ColumnListParameter,
    hidden=HiddenToolParameter,
    hidden_data=HiddenDataToolParameter,
    baseurl=BaseURLToolParameter,
    file=FileToolParameter,
    ftpfile=FTPFileToolParameter,
    genomespacefile=GenomespaceFileToolParameter,
    data=DataToolParameter,
    data_collection=DataCollectionToolParameter,
    library_data=LibraryDatasetToolParameter,
    rules=RulesListToolParameter,
    field=FieldTypeToolParameter,
    drill_down=DrillDownSelectToolParameter
)
#directory=DataToolParameter
#directory=FileToolParameter
```

#### Add missing mapping between Galaxy type and CWL type.

Copy simple_value() func from to_cwl_job() to galactic_flavored_to_cwl_job()

TODO: merge both simple_value func as they are the same ?

```
    def galactic_flavored_to_cwl_job(tool, param_dict, local_working_directory):
    
        def simple_value(input, param_dict_value, type_representation_name=None):
            type_representation = type_representation_from_name(type_representation_name)

            ...
```

Prevent empty string

```
    def exec_before_job(self, app, inp_data, out_data, param_dict=None):

        ...

        input_json = {k:v for k, v in input_json.iteritems() if v != ''}

        ...
```

64f6b95  

#### Prevent unset optional file to trigger 'ValidationException' exception

Exception

```
Traceback (most recent call last):
  File "lib/galaxy/jobs/runners/__init__.py", line 192, in prepare_job
    job_wrapper.prepare()
  File "lib/galaxy/jobs/__init__.py", line 869, in prepare
    tool_evaluator.set_compute_environment(compute_environment, get_special=get_special)
  File "lib/galaxy/tools/evaluation.py", line 118, in set_compute_environment
    self.tool.exec_before_job(self.app, inp_data, out_data, param_dict)
  File "lib/galaxy/tools/__init__.py", line 2421, in exec_before_job
    cwl_command_line = cwl_job_proxy.command_line
  File "lib/galaxy/tools/cwl/parser.py", line 445, in command_line
    if self.is_command_line_job:
  File "lib/galaxy/tools/cwl/parser.py", line 365, in is_command_line_job
    self._ensure_cwl_job_initialized()
  File "lib/galaxy/tools/cwl/parser.py", line 381, in _ensure_cwl_job_initialized
    beta_relaxed_fmt_check=beta_relaxed_fmt_check,
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/cwltool/command_line_tool.py", line 375, in job
    builder.pathmapper = self.makePathMapper(reffiles, builder.stagedir, **make_path_mapper_kwargs)
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/cwltool/command_line_tool.py", line 233, in makePathMapper
    separateDirs=kwargs.get("separateDirs", True))
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/cwltool/pathmapper.py", line 229, in __init__
    self.setup(dedup(referenced_files), basedir)
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/cwltool/pathmapper.py", line 282, in setup
    self.visit(fob, stagedir, basedir, copy=fob.get("writable"), staged=True)
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/cwltool/pathmapper.py", line 271, in visit
    self.visitlisting(obj.get("secondaryFiles", []), stagedir, basedir, copy=copy, staged=staged)
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/schema_salad/sourceline.py", line 164, in __exit__
    raise self.makeError(six.text_type(exc_value))
ValidationException: [Errno 2] No such file or directory: '/home/jra001k/snapshot/galaxy/database/jobs_directory/000/31/None'
```

Fix

```
    def exec_before_job(self, app, inp_data, out_data, param_dict=None):

        ...

        input_json = {k:v for k, v in input_json.iteritems() if not (isinstance(v, dict) and v['class'] == 'File' and v['location'] == 'None')}

        cwl_job_proxy = self._cwl_tool_proxy.job_proxy(
            input_json,
            output_dict,
            local_working_directory,
        )
```

2e55c1c  

#### Set 'beta_relaxed_fmt_check' to prevent file fmt check.

Prevent this exception to occur

```
galaxy.jobs.runners ERROR 2018-07-24 18:10:49,615 [p:5512,w:1,m:0] [LocalRunner.work_thread-0] (192) Failure preparing job
Traceback (most recent call last):
  File "lib/galaxy/jobs/runners/__init__.py", line 192, in prepare_job
    job_wrapper.prepare()
  File "lib/galaxy/jobs/__init__.py", line 869, in prepare
    tool_evaluator.set_compute_environment(compute_environment, get_special=get_special)
  File "lib/galaxy/tools/evaluation.py", line 118, in set_compute_environment
    self.tool.exec_before_job(self.app, inp_data, out_data, param_dict)
  File "lib/galaxy/tools/__init__.py", line 2421, in exec_before_job
    cwl_command_line = cwl_job_proxy.command_line
  File "lib/galaxy/tools/cwl/parser.py", line 443, in command_line
    if self.is_command_line_job:
  File "lib/galaxy/tools/cwl/parser.py", line 364, in is_command_line_job
    self._ensure_cwl_job_initialized()
  File "lib/galaxy/tools/cwl/parser.py", line 379, in _ensure_cwl_job_initialized
    use_container=False,
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/cwltool/command_line_tool.py", line 342, in job 
    builder = self._init_job(job_order, **kwargs)
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/cwltool/process.py", line 641, in _init_job
    builder.bindings.extend(builder.bind_input(self.inputs_record_schema, builder.job, discover_secondaryFiles=kwargs.get("toplevel")))
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/cwltool/builder.py", line 182, in bind_input
    bindings.extend(self.bind_input(f, datum[f["name"]], lead_pos=lead_pos, tail_pos=f["name"], discover_secondaryFiles=discover_secondaryFiles))
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/cwltool/builder.py", line 242, in bind_input
    raise WorkflowException("Expected value of '%s' to have format %s but\n  %s" % (schema["name"], schema["format"], ve))
WorkflowException: Expected value of 'inputRefDBFile' to have format http://edamontology.org/format_1929 but
  File has no 'format' defined: {
    "basename": "uniref90_subset.fasta",
    "nameroot": "uniref90_subset",
    "nameext": ".fasta",
    "location": "/home/jra001k/snapshot/galaxy_h_clone/database/files/000/dataset_205.dat",
    "class": "File",
    "size": 4753
}
```

Fix in lib/galaxy/tools/cwl/cwltool_deps.py

```
beta_relaxed_fmt_check = False # if set to true, file format checking is not perfomed.
```

Fix in lib/galaxy/tools/cwl/parser.py

```
    def _ensure_cwl_job_initialized(self):
        if self._cwl_job is None:

            self._cwl_job = next(self._tool_proxy._tool.job(
                self._input_dict,
                self._output_callback,
                basedir=self._job_directory,
                select_resources=self._select_resources,
                outdir=os.path.join(self._job_directory, "working"),
                tmpdir=os.path.join(self._job_directory, "cwltmp"),
                stagedir=os.path.join(self._job_directory, "cwlstagedir"),
                use_container=False,
                beta_relaxed_fmt_check=beta_relaxed_fmt_check,
            ))
```

2956b44

#### Prevent call to get_size() when dataset is None.

Error

```
galaxy.jobs.runners ERROR 2018-07-24 09:00:41,003 [p:27394,w:1,m:0] [LocalRunner.work_thread-0] (174) Failure preparing job
Traceback (most recent call last):
  File "lib/galaxy/jobs/runners/__init__.py", line 192, in prepare_job    
    job_wrapper.prepare()
  File "lib/galaxy/jobs/__init__.py", line 869, in prepare
    tool_evaluator.set_compute_environment(compute_environment, get_special=get_special)
  File "lib/galaxy/tools/evaluation.py", line 118, in set_compute_environment
    self.tool.exec_before_job(self.app, inp_data, out_data, param_dict)
  File "lib/galaxy/tools/__init__.py", line 2407, in exec_before_job
    input_json = self.param_dict_to_cwl_inputs(param_dict, local_working_directory)
  File "lib/galaxy/tools/__init__.py", line 2482, in param_dict_to_cwl_inputs
    input_json = galactic_flavored_to_cwl_job(self, param_dict, local_working_directory)
  File "lib/galaxy/tools/cwl/representation.py", line 216, in galactic_flavored_to_cwl_job
    inputs_at_depth[map_to] = dataset_wrapper_to_file_json(inputs_dir, param_dict[input_name])
  File "lib/galaxy/tools/cwl/representation.py", line 163, in dataset_wrapper_to_file_json
    raw_file_object["size"] = int(dataset_wrapper.get_size())

TypeError: 
'SafeStringWrapper(str:<class 'galaxy.tools.wrappers.ToolParameterValueWrapper'>, 
<class 'galaxy.util.object_wrapper.SafeStringWrapper'>, <class 'numbers>, <type 'NoneType'>, 
<type 'NotImplementedT' object is not callable
```

Fix

```
def dataset_wrapper_to_file_json(inputs_dir, dataset_wrapper):

    ...

    raw_file_object["location"] = path

    if not isinstance(dataset_wrapper.unsanitized, NoneDataset):
        raw_file_object["size"] = int(dataset_wrapper.get_size())
```

1657c6d

2d2ec56 (duplicate)

### Cwltool

#### Add 'beta_relaxed_fmt_check' to prevent file fmt check.

Prevent this exception to occur

```
galaxy.jobs.runners ERROR 2018-07-24 18:10:49,615 [p:5512,w:1,m:0] [LocalRunner.work_thread-0] (192) Failure preparing job
Traceback (most recent call last):
  File "lib/galaxy/jobs/runners/__init__.py", line 192, in prepare_job
    job_wrapper.prepare()
  File "lib/galaxy/jobs/__init__.py", line 869, in prepare
    tool_evaluator.set_compute_environment(compute_environment, get_special=get_special)
  File "lib/galaxy/tools/evaluation.py", line 118, in set_compute_environment
    self.tool.exec_before_job(self.app, inp_data, out_data, param_dict)
  File "lib/galaxy/tools/__init__.py", line 2421, in exec_before_job
    cwl_command_line = cwl_job_proxy.command_line
  File "lib/galaxy/tools/cwl/parser.py", line 443, in command_line
    if self.is_command_line_job:
  File "lib/galaxy/tools/cwl/parser.py", line 364, in is_command_line_job
    self._ensure_cwl_job_initialized()
  File "lib/galaxy/tools/cwl/parser.py", line 379, in _ensure_cwl_job_initialized
    use_container=False,
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/cwltool/command_line_tool.py", line 342, in job 
    builder = self._init_job(job_order, **kwargs)
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/cwltool/process.py", line 641, in _init_job
    builder.bindings.extend(builder.bind_input(self.inputs_record_schema, builder.job, discover_secondaryFiles=kwargs.get("toplevel")))
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/cwltool/builder.py", line 182, in bind_input
    bindings.extend(self.bind_input(f, datum[f["name"]], lead_pos=lead_pos, tail_pos=f["name"], discover_secondaryFiles=discover_secondaryFiles))
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/cwltool/builder.py", line 242, in bind_input
    raise WorkflowException("Expected value of '%s' to have format %s but\n  %s" % (schema["name"], schema["format"], ve))
WorkflowException: Expected value of 'inputRefDBFile' to have format http://edamontology.org/format_1929 but
  File has no 'format' defined: {
    "basename": "uniref90_subset.fasta",
    "nameroot": "uniref90_subset",
    "nameext": ".fasta",
    "location": "/home/jra001k/snapshot/galaxy_h_clone/database/files/000/dataset_205.dat",
    "class": "File",
    "size": 4753
}
```

Fix

```
def arg_parser():  # type: () -> argparse.ArgumentParser
    parser = argparse.ArgumentParser(
        description='Reference executor for Common Workflow Language standards.')
    
    ...

    parser.add_argument("--beta_relaxed_fmt_check", action="store_true", help="Disable file format validation.")

    ...
```

```
def bind_input(self, schema, datum, lead_pos=None, tail_pos=None, discover_secondaryFiles=False):   

    ...

    if "format" in schema and not self.beta_relaxed_fmt_check:
        try:
            checkFormat(datum, self.do_eval(schema["format"]), self.formatgraph)
        except validate.ValidationException as ve:
            raise WorkflowException("Expected value of '%s' to have format %s but\n  %s" % (schema["name"], schema["format"], ve))

    ...
```

b02b33f

82b0d0c

### ELIXIR Workflow

#### Add 'gx:interface' hints in tools CWL files

Example with BUSCO tools

```
hints:
  - class: gx:interface
    gx:inputs:
      - gx:name: blastSingleCore
        gx:type: boolean
        gx:optional: True
      - gx:name: cpu
        gx:type: integer
        gx:optional: True
      - gx:name: evalue
        gx:type: float
        gx:optional: True
      - gx:name: force
        gx:type: boolean
        gx:optional: True
      - gx:name: help
        gx:type: boolean
        gx:optional: True
      - gx:name: lineage
        gx:type: data
      - gx:name: long
        gx:type: boolean
        gx:optional: True
      - gx:name: mode
        gx:value: tran
        gx:type: text
      - gx:name: outputName
        gx:value: TEST
        gx:type: text
      - gx:name: quiet
        gx:type: boolean
        gx:optional: True
      - gx:name: regionLimit
        gx:type: integer
        gx:optional: True
      - gx:name: restart
        gx:type: boolean
        gx:optional: True
      - gx:name: sequenceFile
        gx:format: 'txt'
        gx:type: data
      - gx:name: species
        gx:type: text
        gx:optional: True
      - gx:name: tarzip
        gx:type: boolean
        gx:optional: True
      - gx:name: tempPath
        gx:type: data
        gx:optional: True
      - gx:name: version
        gx:type: boolean
        gx:optional: True
```

92ee2b9 

2d96dbc

7d0a7ad

abc3d8f 

d9d1610 

e5cb9a7 

5e11714 

fc5ab08 

a1dd63a 

#### Map tar file to Directory CWL type.

Use "gx:type: data" in hints section for Directory CWL type, so we
have the right control in the Galaxy tools form (i.e. to select the
tar file).

```
hints:
  - class: gx:interface
    gx:inputs:
      - gx:name: tempPath
        gx:type: data
        gx:optional: True
```

8fcf887

#### Temporary fix for search_space_size error.

Required int parameter (i.e. non optional) cause error below

```
galaxy.jobs.runners ERROR 2018-07-24 17:10:22,683 [p:2318,w:1,m:0] [LocalRunner.work_thread-0] (187) Failure preparing job
Traceback (most recent call last):
  File "lib/galaxy/jobs/runners/__init__.py", line 192, in prepare_job
    job_wrapper.prepare()
  File "lib/galaxy/jobs/__init__.py", line 869, in prepare
    tool_evaluator.set_compute_environment(compute_environment, get_special=get_special)
  File "lib/galaxy/tools/evaluation.py", line 118, in set_compute_environment
    self.tool.exec_before_job(self.app, inp_data, out_data, param_dict)
  File "lib/galaxy/tools/__init__.py", line 2419, in exec_before_job
    local_working_directory,
  File "lib/galaxy/tools/cwl/parser.py", line 221, in job_proxy
    return JobProxy(self, input_dict, output_dict, job_directory=job_directory)
  File "lib/galaxy/tools/cwl/parser.py", line 356, in __init__
    self._normalize_job()
  File "lib/galaxy/tools/cwl/parser.py", line 389, in _normalize_job
    process.fillInDefaults(self._tool_proxy._tool.tool["inputs"], self._input_dict)
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/cwltool/process.py", line 365, in fillInDefaults
    raise WorkflowException("Missing required input parameter '%s'" % shortname(inp["id"]))
  File "/home/jra001k/snapshot/galaxy_h_clone/.venv/local/lib/python2.7/site-packages/schema_salad/sourceline.py", line 164, in __exit__
    raise self.makeError(six.text_type(exc_value))
WorkflowException: ../workflow-is-cwl_h/tools/Infernal/cmsearch/infernal-cmsearch-v1.1.2.cwl:54:5: Missing required input parameter 'search_space_size'
```

```
  - id: search_space_size
    type: int
    inputBinding:
      position: 0
      prefix: '-Z'
```

Fix

```
  - id: search_space_size
    label: search space size in *Mb* to <x> for E-value calculations
    type: int?
    inputBinding:
      position: 0
      prefix: '-Z'
    label: search space size in *Mb* to <x> for E-value calculations
```

db248cd

#### Replace relative paths with absolute paths.

a876284  

#### Modify label to extract tools name from Galaxy-CWL (#1).

8096119  

## Problems that remain

* Tools default values not set when running a workflow (must be set manually)
* Tools need to be created in galaxy before importing and running a CWL workflow
(this is strange as dynamic tools are created on-the-fly when importing the workflow. This need further investigation.)
* Find alternative to replacing relative-path with absolute-path in CWL workflow files (CWL-pack ?)
* Find alternative to adding Directory type support using tar file (as it prevents tar files from being used for another purpose in Galaxy)
* At Galaxy startup, tools are loaded twice in order to populate '_tools_by_hash' variable.
* After the workflow run, the browser sends requests to Galaxy server without interruption (every second).
* Workflow import take a long time (maybe caused by the creation of each
  dynamic tools on-the-fly). Maybe a progress bar is needed to inform the user
  that the import process is running.
* Find alternative to assuming CWL workflow if exception occurs during json
  deserialization (when user click on 'import'). This is problematic as Galaxy
  native workflow may use yaml too in the future.


## DEMO server

* Url: http://sd-104052.dedibox.fr:8083
* Authentication login: democwl
* Galaxy demo user: demo-g-cwl-is.test

The DEMO server contains all the modifications described in this document and
can successfully import and run the following workflow: https://github.com/hmenager/workflow-is-cwl/workflows/cmsearch-multimodel-wf.cwl

<!-- 
# vim: tw=70
-->
