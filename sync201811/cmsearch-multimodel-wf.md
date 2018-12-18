# Running *cmsearch-multimodel-wf* in Galaxy-CWL

## Overview

This document describes workflow test using last version of
https://github.com/common-workflow-language/galaxy

Last commit synced: 9899eb1cfa6832f5058294e39a5d20380bb526c8

## Fork used

Fix proposals are committed on repositories below:

```
https://github.com/hmenager/galaxy
```

```
https://github.com/jra001k/gxformat2
```

## Modifications description

### Fix TypeError exception when importing a packed workflow.

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

This is because 'or' operator interprets 0 value as false.

Replacing 'or' with else/if block fixes the problem.

f68e787f218531dc3d106b81b7d296bf1822d125
(gxformat2 fork)

### foobar3

# vim: tw=70
