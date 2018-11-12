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

## Modifications overview

* Add *gx:interface* hints in CWL tools files
* Prevent filetype checking (EDAM) in Cwltool
* Add *Directory* type support in Galaxy (using binding between tar-file and cwl-directory-type)
* Assume yaml CWL workflow if exception occurs during json deserialization (when user click on 'import')
* Replace *relative path* with *absolute path* in *run* attributes of CWL workflow files
* Call '_init_dynamic_tools' method at Galaxy startup to populate '_tools_by_hash' variable

## Modifications description

FIXME

## Modifications detailed description

Code modification details are brought together in this document

<https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_code.md>

## Problems that remain

* Tools default values not set when running a workflow (must be set manually)
* Tools need to be created in galaxy before importing and running a CWL workflow (this is strange as dynamic tools are created on-the-fly when importing the workflow. This need further investigation.)
* Find alternative to replacing relative-path with absolute-path in CWL workflow files (CWL-pack ?)
* Find alternative to adding Directory type support using tar file (as it prevents tar file from being used for another purpose in Galaxy)
* At Galaxy startup, the fix to populate '_tools_by_hash' variable seems not a good solution as it consumes a lot of memory and CPU. This need further investigation.
* Once the run is complete, the browser keep sending requests to Galaxy server without interruption (every second).
* Workflow import take a long time (maybe caused by the on-the-fly creation of each dynamic tool).
* During the *Workflow import*, the message "Import failed for unkown reason" is sometime displayed on the client side (without disrupting the import process).
* Find alternative to assuming CWL workflow if exception occurs during json deserialization. This is problematic as Galaxy native workflow may also use yaml in the future.

## Demo server

The demo server contains all the modifications described in this
document and can successfully import and run the following workflow:
https://github.com/hmenager/workflow-is-cwl/blob/master/workflows/cmsearch-multimodel-wf.cwl

* Url: http://sd-104052.dedibox.fr:8083
* Authentication login: democwl
* Galaxy demo user: demo@g-cwl-is.test

## Annexes

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
