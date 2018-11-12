# Running *cmsearch-multimodel-wf* in Galaxy-CWL

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

* Prevent EDAM filetype checking in Cwltool
* Add 'Directory' type support in Galaxy (using binding between tar-file and directory-type)
* Add 'gx:interface' hints in CWL tools files
* Replace relative-path with absolute-path in 'run' attributes of CWL workflow files (tools links).
* Call '_init_dynamic_tools' method at Galaxy startup to populate '_tools_by_hash' variable.
* Assume yaml CWL workflow if exception occurs during json deserialization (when user click on 'import')

## Modifications description

FIXME

## Modifications detailed description

Code modification details are brought together in this document

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md>

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

## Demo server

The demo server contains all the modifications described in this
document and can successfully import and run the following workflow:
https://github.com/hmenager/workflow-is-cwl/workflows/cmsearch-multimodel-wf.cwl

* Url: http://sd-104052.dedibox.fr:8083
* Authentication login: democwl
* Galaxy demo user: demo-g-cwl-is.test

## Annex

Annex 1

Code modification details

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md>

Annex 2

Demo server installation guide

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_demosrvinst.md>

Annex 3

Demo procedure

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_demo.md>

<!-- 
# vim: tw=70
-->
