# Running *cmsearch-multimodel-wf* in Galaxy-CWL

## Overview

This document describes CWL workflow test with Galaxy using last version of
https://github.com/common-workflow-language/galaxy

When this document has been written, HEAD commit  was 9899eb1cfa6832f5058294e39a5d20380bb526c8

## Fork

Fix proposals are committed in repositories below:

```
https://github.com/hmenager/galaxy
```

```
https://github.com/hmenager/gxformat2
```

```
https://github.com/hmenager/workflow-is-cwl
```

## Modifications description

### TypeError exception when importing a packed workflow.

[Workflow](https://github.com/hmenager/workflow-is-cwl/blob/master/workflows/cmsearch-multimodel-wf-packed.cwl) is packed with command below:

```
$ cwltool --pack cmsearch-multimodel-wf.cwl > cmsearch-multimodel-wf-packed.cwl
```
 
When importing the packed workflow in Galaxy, exception below occurs:

```
Traceback (most recent call last):
  File "lib/galaxy/web/framework/decorators.py", line 283, in decorator
    rval = func(self, trans, *args, **kwargs)
  File "lib/galaxy/webapps/galaxy/api/workflows.py", line 337, in create
    return self.__api_import_from_archive(trans, archive_data, source="uploaded file", from_path=os.path.abspath(uploaded_file_name))
  File "lib/galaxy/webapps/galaxy/api/workflows.py", line 596, in __api_import_from_archive
    raw_workflow_description = self.__normalize_workflow(trans, data)
  File "lib/galaxy/webapps/galaxy/api/workflows.py", line 679, in __normalize_workflow
    return self.workflow_contents_manager.normalize_workflow_format(trans, as_dict)
  File "lib/galaxy/managers/workflows.py", line 315, in normalize_workflow_format
    workflow_path += "#" + object_id
TypeError: unsupported operand type(s) for +=: 'NoneType' and 'str'
```

Adding 'src' and 'path' attributes in 'data' dict prevents the exception.

e6a4d11670c9e22dba5e410a2899cfeafccdbadd
([galaxy fork](https://github.com/hmenager/galaxy))

### KeyError exception when saving a packed workflow

When importing a packed workflow, some output_id use 0 value, which
causes the exception below (when we click on "Save" in workflow editor
to persist modifications):

```
Traceback (most recent call last):
  File "lib/galaxy/web/framework/decorators.py", line 283, in decorator
    rval = func(self, trans, *args, **kwargs)
  File "lib/galaxy/webapps/galaxy/api/workflows.py", line 552, in update
    **from_dict_kwds
  File "lib/galaxy/managers/workflows.py", line 425, in update_workflow_from_raw_description
    self._sync_stored_workflow(trans, stored_workflow)
  File "lib/galaxy/managers/workflows.py", line 540, in _sync_stored_workflow
    wf_dict = from_galaxy_native(wf_dict, None, json_wrapper=True)
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/gxformat2/export.py", line 48, in from_galaxy_native
    source = _to_source(workflow_output, label_map, output_id=step["id"])
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/gxformat2/export.py", line 217, in _to_source
    output_id = output_id or has_output_name['id']
KeyError: 'id'
```

It seems caused by 'or' operator interpreting 0 value as false.

Replacing 'or' with an 'else/if' block fixes the problem.

f68e787f218531dc3d106b81b7d296bf1822d125
([gxformat2 fork](https://github.com/hmenager/gxformat2))

### Map tar file to 'Directory' type

When running the BUSCO tool, error below occurs

```
Traceback (most recent call last):
  File "lib/galaxy/jobs/runners/__init__.py", line 214, in prepare_job
    job_wrapper.prepare()
  File "lib/galaxy/jobs/__init__.py", line 871, in prepare
    tool_evaluator.set_compute_environment(compute_environment, get_special=get_special)
  File "lib/galaxy/tools/evaluation.py", line 123, in set_compute_environment
    self.tool.exec_before_job(self.app, inp_data, out_data, param_dict)
  File "lib/galaxy/tools/__init__.py", line 2457, in exec_before_job
    cwl_command_line = cwl_job_proxy.command_line
  File "lib/galaxy/tools/cwl/parser.py", line 516, in command_line
    if self.is_command_line_job:
  File "lib/galaxy/tools/cwl/parser.py", line 428, in is_command_line_job
    self._ensure_cwl_job_initialized()
  File "lib/galaxy/tools/cwl/parser.py", line 452, in _ensure_cwl_job_initialized
    *args, **kwargs
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/cwltool/command_line_tool.py", line 342, in job
    builder = self._init_job(job_order, **kwargs)
  File "/home/jra001k/snapshot/galaxy/.venv/local/lib/python2.7/site-packages/cwltool/process.py", line 594, in _init_job
    raise WorkflowException("Invalid job input record:\n" + Text(e))
WorkflowException: Invalid job input record:
the `lineage` field is not valid because
  Expected class 'Directory' but this is 'File'
```

The two commits below fix this error:

a4b82fa75a5fdb79aba13410cc9a611f593b07b4

3399e035cdc98be41feb5249e6d9ee64fd40e4a3

([galaxy fork](https://github.com/hmenager/galaxy))

Note: this patch is a rewriting of a
[previous try](https://github.com/common-workflow-language/galaxy/commit/6e675f33b93c08d939fd520ecfafb6ee8a9726df),
which should prevent regression in "Directory output test".

### The tool 'tmpugCygk#infernal-cmsearch-v1.1.2.cwl' is missing. Cannot build workflow module.

When we restart Galaxy (after having edited and saved the workflow),
and we try to edit the wf again,  warning below occurs

```
galaxy.workflow.modules WARNING The tool 'tmpugCygk#infernal-cmsearch-v1.1.2.cwl' is missing. Cannot build workflow module.
galaxy.workflow.modules WARNING The tool 'tmpugCygk#concatenate.cwl' is missing. Cannot build workflow module.
galaxy.workflow.modules WARNING The tool 'tmpugCygk#cmsearch-deoverlap-v0.02.cwl' is missing. Cannot build workflow module.
```

then the workflow cannot be edited anymore (links and boxes are missing).

It seems to be related by the following change in the database:

Before saving
```
sqlite> select type,label,tool_id,tool_hash from workflow_step where type='tool' order by label; 
tool        cmsearch                7943013e61897f4949e77546edeabd960a2f608067e2ecd523a1d2ef010e9682
tool        concatenat              742a9252f4cf1b80bcb57d20aa4133edbaa46b8a3112a9055110db18b1e446dc
tool        remove_ove              e384c3b53464dbea34880afedfb09f6857ab89313014f76f9eea23e15e1c83b8
```

After saving
```
sqlite> select type,label,tool_id,tool_hash from workflow_step where type='tool' order by label; 
tool        cmsearch                   7943013e61897f4949e77546edeabd960a2f608067e2ecd523a1d2ef010e9682
tool        cmsearch    tmpkadxtU#inf                                                                  
tool        concatenat                 742a9252f4cf1b80bcb57d20aa4133edbaa46b8a3112a9055110db18b1e446dc
tool        concatenat  tmpkadxtU#con                                                                  
tool        remove_ove                 e384c3b53464dbea34880afedfb09f6857ab89313014f76f9eea23e15e1c83b8
tool        remove_ove  tmpkadxtU#cms  
```

(after clicking on 'Save' button in the workflow editor, tools are
referenced using 'tool_id' column instead of 'tool_hash' column (in
'workflow_step' table)).

When hacking the code 
([link](https://github.com/hmenager/galaxy/commit/3c167c3f8e945d67190e5faacc8fa1b14ce9e194)) 
to force populating 'tool_hash' column and setting
'tool_id' column to NULL, the problem seems to disappear.
