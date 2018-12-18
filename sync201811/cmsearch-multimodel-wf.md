# Running *cmsearch-multimodel-wf* in Galaxy-CWL

## Overview

This document describes workflow test using last version of
https://github.com/common-workflow-language/galaxy

HEAD commit when this document has been written: 9899eb1cfa6832f5058294e39a5d20380bb526c8

## Fork used

Fix proposals are committed in repositories below:

```
https://github.com/hmenager/galaxy
```

```
https://github.com/jra001k/gxformat2
```

```
https://github.com/hmenager/workflow-is-cwl
```

## Modifications description

### TypeError exception when importing a packed workflow.

Workflow is packed with command below:

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
(Galaxy fork)

### Add support for 0 value in output_id

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

Replacing 'or' with else/if block fixes the problem.

f68e787f218531dc3d106b81b7d296bf1822d125
(gxformat2 fork)

### 

When we restart Galaxy (after having edited and saved the workflow),
and we try to edit the wf again,  warning below occurs

```
galaxy.workflow.modules WARNING 2018-12-11 06:15:52,195 [p:16652,w:1,m:0] [uWSGIWorker1Core0] The tool 'tmpugCygk#infernal-cmsearch-v1.1.2.cwl' is missing. Cannot build workflow module.                                       
galaxy.workflow.modules WARNING 2018-12-11 06:15:52,210 [p:16652,w:1,m:0] [uWSGIWorker1Core0] The tool 'tmpugCygk#concatenate.cwl' is missing. Cannot build workflow module.                                                    
galaxy.workflow.modules WARNING 2018-12-11 06:15:52,214 [p:16652,w:1,m:0] [uWSGIWorker1Core0] The tool 'tmpugCygk#cmsearch-deoverlap-v0.02.cwl' is missing. Cannot build workflow module.                                       
```

then the workflow cannot be edited anymore (links and boxes are missing).

It seems to be related by the following change in the database:

after clicking on 'Save' button in the workflow editor, tool are now
referenced using 'tool_id' column (e.g.
'tmpPd1pDz#cmsearch-deoverlap-v0.02.cwl') instead of 'tool_hash'
column (in 'workflow_step' table).

e.g.

Before saving:
```
sqlite> select type,label,tool_id,tool_hash from workflow_step where type='tool' order by label; 
tool        cmsearch                7943013e61897f4949e77546edeabd960a2f608067e2ecd523a1d2ef010e9682
tool        concatenat              742a9252f4cf1b80bcb57d20aa4133edbaa46b8a3112a9055110db18b1e446dc
tool        remove_ove              e384c3b53464dbea34880afedfb09f6857ab89313014f76f9eea23e15e1c83b8
```
After saving:
```
sqlite> select type,label,tool_id,tool_hash from workflow_step where type='tool' order by label; 
tool        cmsearch                7943013e61897f4949e77546edeabd960a2f608067e2ecd523a1d2ef010e9682
tool        cmsearch    tmpkadxtU#                                                                  
tool        concatenat              742a9252f4cf1b80bcb57d20aa4133edbaa46b8a3112a9055110db18b1e446dc
tool        concatenat  tmpkadxtU#                                                                  
tool        remove_ove              e384c3b53464dbea34880afedfb09f6857ab89313014f76f9eea23e15e1c83b8
tool        remove_ove  tmpkadxtU#  
```

# vim: tw=70
