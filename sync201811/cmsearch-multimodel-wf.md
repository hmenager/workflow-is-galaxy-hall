# Running *cmsearch-multimodel-wf* in Galaxy-CWL

## Overview

This document describes wf execution tests
using last version of https://github.com/common-workflow-language/galaxy

## Fork used

https://github.com/hmenager/galaxy

https://github.com/jra001k/gxformat2

## Modifications summary

* Add *gx:interface* hints in CWL tools files
* Prevent filetype checking (EDAM) in Cwltool
* Add *Directory* type support in Galaxy (using binding between tar-file and cwl-directory-type)

## Modifications description

1. [foo1](#foobar1)
2. [foo2](#foobar2)
3. [foo3](#foobar3)

### foobar1

*gx:interface* allows to manually bind a CWL type to a Galaxy type for a given parameter in the Tools CWL file.

Currently, before runnning a CWL workflow in Galaxy, all CWL tools used by the
workflow must be manually modified to add those bindings.

### foobar2

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

### foobar3

# vim: tw=70
