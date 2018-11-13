# Running *cmsearch-multimodel-wf* in Galaxy-CWL

## Overview

*cmsearch-multimodel-wf* is a CWL workflow part of ELIXIR project.

In order to run *cmsearch-multimodel-wf* workflow with Galaxy-CWL, some
modifications must be made in software and CWL files below:

* Galaxy-CWL 
* Cwltool 
* CWL tools files
* CWL workflow files

This document describes those modifications.

## Repositories used

Galaxy-CWL

<https://github.com/hmenager/galaxy>

Cwltool

<https://github.com/hmenager/cwltool>

ELIXIR Workflow

<https://github.com/hmenager/workflow-is-cwl>

## Main modifications

* Add *gx:interface* hints in CWL tools files
* Prevent filetype checking (EDAM) in Cwltool
* Add *Directory* type support in Galaxy (using binding between tar-file and cwl-directory-type)
* Replace *relative path* with *absolute path* in *run* attributes of CWL workflow files
* Call '_init_dynamic_tools' method at Galaxy startup to populate '_tools_by_hash' variable

## Modifications description

1. [Add *gx:interface* hints in CWL files](#add-gxinterface-hints-in-cwl-files)
2. [*required int parameter* not working](#required-int-parameter-not-working)
3. [Prevent flooding Galaxy left panel with large Tools label](#prevent-flooding-galaxy-left-panel-with-large-tools-label)
4. [Map tar file to *Directory* CWL type](#map-tar-file-to-directory-cwl-type)
5. [Add missing mappings between Galaxy type and CWL type](#add-missing-mappings-between-galaxy-type-and-cwl-type)
6. [Prevent *unset optional file* to trigger *ValidationException*](#prevent-unset-optional-file-to-trigger-validationexception)
7. [Add *beta_relaxed_fmt_check* to prevent file format check](#add-beta_relaxed_fmt_check-to-prevent-file-format-check)
8. [Prevent call to get_size() when dataset is None](#prevent-call-to-get_size-when-dataset-is-none)
9. [Rename *Test Dataset*](#rename-test-dataset)
10. [Enable CWL workflow import with GUI](#enable-cwl-workflow-import-with-gui)
11. [Replace relative paths with absolute paths](#replace-relative-paths-with-absolute-paths)
12. [Enable CWL workflow execution with GUI](#enable-cwl-workflow-execution-with-gui)

### Add *gx:interface* hints in CWL files

*gx:interface* allows to manually bind a CWL type to a Galaxy type for a given parameter in the Tools CWL file.

Currently, before runnning a CWL workflow in Galaxy, all CWL tools used by the
workflow must be manually modified to add those bindings.

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#add-gxinterface-hints-in-cwl-files)

### *required int parameter* not working

Required int parameter (i.e. non optional) cause error below

```
  - id: search_space_size
    type: int
```

```
WorkflowException: ../workflow-is-cwl_h/tools/Infernal/cmsearch/infernal-cmsearch-v1.1.2.cwl:54:5: Missing required input parameter 'search_space_size'
```

As a temporary fix, the Tools CWL file is modified to set the parameter as optional.

```
  - id: search_space_size
    type: int?
```

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#required-int-parameter-not-working)

### Prevent flooding Galaxy left panel with large Tools label

Tools *label* in CWL file is too long (up to ten words) to be used in Galaxy
left panel to identify the tool.

To circumvent this problem, a special format is used in Tools *label* attribute,
in order to include a short identifier which can be easily extracted.

This short name is then displayed in Galaxy left panel to identify the tool.

Example

```
label: Remove lower scoring overlaps from cmsearch --tblout files.                                                                                                                                                                 
```

become

```
label: >-
  Cmsearch-deoverlap: Remove lower scoring overlaps from cmsearch --tblout files.                                                                                                                                                  
```

In this example, the short name is 'Cmsearch-deoverlap'.

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#prevent-flooding-galaxy-left-panel-with-large-tools-label)

### Map tar file to *Directory* CWL type

Directory type does not exist in Galaxy.

To be able to run CWL tools which use Directory CWL type, some modifications
must be made in Galaxy-CWL code.

A temporary hack has been set up:

    * Tools parameters with Directory type are binded to the *data* Galaxy type
      using Tools hints section (gx:interface). This will provide the 
      *Data Selector* control in the Galaxy Tools Form.
    * A tar file with the directory content is created.
    * The tar file is uploaded in Galaxy history.
    * To execute the Tools, the tar file must be used in place of the Directory.
    * During the run, the tar file is automatically extracted by Galaxy 
      and a directory is provided to Cwltool.

Note: this hack has serious limitation as it prevents using file with 'tar'
extension for another purpose than wrapping a directory.

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#map-tar-file-to-directory-cwl-type)

### Add missing mappings between Galaxy type and CWL type

BUSCO Tool execution fails because some CWL parameter types (e.g. string,
enum) are not converted to Galaxy type.

This is because types mapping are correctly set in to_cwl_job(), but not in
galactic_flavored_to_cwl_job().

To fix the problem, type mapping code has been duplicated from to_cwl_job() to
galactic_flavored_to_cwl_job().

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#add-missing-mapping-between-galaxy-type-and-cwl-type)

### Prevent *unset optional file* to trigger *ValidationException*

Exception below occurs when running a Tools with unset optional input dataset 

```
ValidationException: [Errno 2] No such file or directory: '/home/jra001k/snapshot/galaxy/database/jobs_directory/000/31/None'
```

To prevent this problem, the code below has been added to exec_before_job()
method. It removes unset dataset from the input dataset list.

```
input_json = {k:v for k, v in input_json.iteritems() if not (isinstance(v, dict) and v['class'] == 'File' and v['location'] == 'None')}
```

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#prevent-unset-optional-file-to-trigger-validationexception)

### Add *beta_relaxed_fmt_check* to prevent file format check

When running CWL tools, some parameters are defined using EDAM format.

Those formats are not currently supported in Galaxy.

When such tools run in Galaxy, error below occurs:

```
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

To prevent this exception to occur, *beta_relaxed_fmt_check* option has been
added in Cwltool to disable EDAM format checking.

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#add-beta_relaxed_fmt_check-to-prevent-file-fmt-check)

### Prevent call to get_size() when dataset is None

When an user run a CWL Tools and an optional input dataset is not set by the
user, the exception below occurs:

```
  File "lib/galaxy/tools/cwl/representation.py", line 163, in dataset_wrapper_to_file_json
    raw_file_object["size"] = int(dataset_wrapper.get_size())

TypeError: 
'SafeStringWrapper(str:<class 'galaxy.tools.wrappers.ToolParameterValueWrapper'>, 
<class 'galaxy.util.object_wrapper.SafeStringWrapper'>, <class 'numbers>, <type 'NoneType'>, 
<type 'NotImplementedT' object is not callable
```

This is because the get_size() method is called on an unexisting dataset.

To fix the problem, the get_size() method is not called is the dataset type is
*NoneDataset*.

```
def dataset_wrapper_to_file_json(inputs_dir, dataset_wrapper):

    ...

    if not isinstance(dataset_wrapper.unsanitized, NoneDataset):
        raw_file_object["size"] = int(dataset_wrapper.get_size())
```

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#prevent-call-to-get_size-when-dataset-is-none)

### Rename *Test Dataset*

When running CwlWorkflowsTestCase.test_simplest_wf test, exception below occurs:

```
WorkflowException: Invalid filename: 'Test Dataset' contains illegal characters
```

This is caused by space character in the dataset name, in *upload_payload*
method in test/base/populators.py.

To prevent the exception, the dataset has been renamed replacing space with underscore.

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#rename-test-dataset)

### Enable CWL workflow import with GUI

When importing a CWL workflow using GUI import tool, we get the exception below

```
raise exceptions.MessageException("The data content does not appear to be a valid workflow.")
```

This is because Galaxy expects the workflow to bee in JSON format, not YAML format.

A hack has been set up in Galaxy (in *__api_import_from_archive* method) to use
*from_path* argument as an alternative route, if a JSON exception occurs.

This hack allows to import a CWL workflow inside Galaxy succesfully.

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#enable-cwl-workflow-import-with-gui)

### Replace relative paths with absolute paths

When importing a CWL workflow in Galaxy, the exception below occurs

```
ValidationException: database/tmp/tmpM55sBv:21:1: checking field `steps`
database/tmp/tmpM55sBv:52:5:   checking object `database/tmp/tmpM55sBv#remove_overlaps`
database/tmp/tmpM55sBv:60:5:     Field `run` contains undefined reference to `file:///home/foobar/snapshot/galaxy/database/tools/cmsearch-deoverlap/cmsearch-deoverlap-v0.02.cwl`
```

It happens because when the workflow is imported, Galaxy have no knowledge of
the directories tree where the linked Tools are stored. It sees only one CWL file.

As a temporary solution, the directories tree with Tools CWL files is copied on
the Galaxy server, and the workflow CWL file is modified before the import to
replace the relative paths with absolute paths (in the *run* attribute).

Example

```
steps:
    run: /home/jra001k/snapshot/pasteur/workflow-is-cwl/tools/Infernal/cmsearch/infernal-cmsearch-v1.1.2.cwl
    #run: ../tools/Infernal/cmsearch/infernal-cmsearch-v1.1.2.cwl
```

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#replace-relative-paths-with-absolute-paths)

### Enable CWL workflow execution with GUI

When executing a CWL workflow in Galaxy, the following exception occurs

```
  File "lib/galaxy/tools/toolbox/base.py", line 439, in get_tool
    tool_id = self._tools_by_hash[tool_hash].id
KeyError: u'e4de79296ec91ab9b8d8d9d71f94044a2561c01b9fc708e5197d432b453fa297'
```

It seems to be caused by the '_tools_by_hash' attribute not being initialized
(to be confirmed)

To fix the problem, a call to _init_dynamic_tools() method have been added
in *UniverseApplication* class constructor.

Also, 'filter' method has been replaced by 'filter_by' method to prevent
exception below:

```
  File "lib/galaxy/managers/tools.py", line 117, in list_tools
    return self.query().filter(active=active)
TypeError: filter() got an unexpected keyword argument 'active'
```

[More info](https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md#enable-cwl-workflow-execution-with-gui)

## Modifications detailed description

Code modification details are brought together in this document

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md>

## Demo server

The Galaxy-CWL demo server contains all the modifications described in this
document and can successfully performs the following tasks on
the *cmsearch-multimodel-wf* CWL workflow:

* Import wf in Galaxy
* View wf in Galaxy Workflow Editor
* Run wf
* Export wf from Galaxy (json format)

### Demo server informations

* Url: http://sd-104052.dedibox.fr:8083
* Authentication login: democwl
* Galaxy demo user: demo@g-cwl-is.test

### Demo server installation guide

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_demosrvinst.md>

### Demo procedure

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_demo.md>

## Problems that remain

* Find an alternative to replacing relative-path with absolute-path in CWL workflow files (CWL-pack ?).
* During the *Workflow import*, the message "Import failed for unkown reason" is sometime displayed on the client side (without disrupting the import process).
* Find an alternative to using substring of Tools label as Tools idenfier to be displayed on the Galaxy left panel.
* Tools default values not set when running a workflow (must be set manually).
* At Galaxy startup, the fix to populate '_tools_by_hash' variable seems not a good solution as it consumes a lot of memory and CPU. This needs further investigation.
* Find alternative to adding Directory type support using tar file (as it prevents tar file from being used for another purpose in Galaxy).
* Workflow import takes a long time (maybe caused by the on-the-fly creation of each dynamic tool ?).
* Once the run is complete, the browser keep sending requests to Galaxy server without interruption (every second).
* Tools need to be created in galaxy before importing and running a CWL workflow (this is strange as dynamic tools are created on-the-fly when importing the workflow. This needs further investigation.).
* When importing a workflow in Galaxy, find alternative to assuming CWL workflow if exception occurs during json deserialization. This is problematic as Galaxy native workflow may also use yaml in the future.

## Annexes

### Annex 1

Code modification details

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md>

### Annex 2

Demo server installation guide

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_demosrvinst.md>

### Annex 3

Demo procedure

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_demo.md>

### Annex 4

*cmsearch-multimodel-wf* Workflow CWL file

https://github.com/hmenager/workflow-is-cwl/blob/master/workflows/cmsearch-multimodel-wf.cwl

<!-- 
# vim: tw=70
-->
