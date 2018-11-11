# Intro

# Documentation

Hackathon Nov 2018

```
https://github.com/elixir-europe/BioHackathon/blob/master/tools/CWL%20support%20in%20Galaxy/README.md
```

```
https://docs.google.com/presentation/d/1zXpOhJDeVYs-J-1jt0rz7gox10o78cFqshiM-wy8Xh0/edit#slide=id.p1
```

Galaxy-CWL Implementation issues

```
https://github.com/common-workflow-language/galaxy/issues
```

Galaxy-CWL tools and workflow issues

```
https://github.com/hmenager/workflow-is-cwl/issues
```

Galaxy-CWL *cmsearch-multimodel-wf* workflow test

```
https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_workflow_test.md
```

```
https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/demosrv_inst.md
```

Documentation Index

```
https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/README.md
```

# Setup

The repository where the Galaxy-CWL work is done is a [fork of the official one](https://github.com/common-workflow-language/galaxy). We (@hmenager, @khhillion, @jra001k) are working on a [sub-fork of it](https://github.com/common-workflow-language/galaxy).

Here's a typical way to set up your environment for development purposes:

```shell
# clone the repo subfork as origin
git clone https://github.com/hmenager/galaxy.git
cd galaxy
# add the upstream repo
git remote add upstream https://github.com/common-workflow-language/galaxy.git
# check the remotes
git remote -v
# prints this
#origin	https://github.com/hmenager/galaxy.git (fetch)
#origin	https://github.com/hmenager/galaxy.git (push)
#upstream	https://github.com/common-workflow-language/galaxy.git (fetch)
#upstream	https://github.com/common-workflow-language/galaxy.git (push)
```

# Updating the repository

Updating the _origin_ repository should be straightforward.

```shell
git pull origin
```

John performs rebases on common-workflow-language/galaxy on a regular basis. When such rebases are performed, the way to update origin is to perform a rebase locally and then send force push it locally

```shell
git rebase -i upstream/cwl-1.0
# opens a text editor containing a list of commits, replace "pick" with "drop" on every commit that is already included in upstream
git push --force
```

# Where are the tests?

The list of CWL conformance tests in Galaxy is located in `test/unit/tools/cwl_tools/v1.0`. This directory contains the list itself, in the `conformance_tests.yaml` file, and the various CWL tools, workflows, and data in there. These files are copied from the [CWL repo](https://github.com/common-workflow-language/common-workflow-language), where they are located in `v1.0/conformance_test_v1.0.yaml` for the list itself and `v1.0/v1.0/` for the dependencies.

# Running the tests

The conformance tests can be run using the Galaxy CWL unit tests with the following command:

```shell
# first download a fairly recent version of the database so that the tests start more quickly, by using an already-migrated version
wget https://github.com/jmchilton/galaxy-downloads/blob/master/db_gx_rev_0141.sqlite?raw=true
export GALAXY_TEST_DB_TEMPLATE=db_gx_rev_0141.sqlite
# run all the tests
./run_tests.sh -api test/api/test_cwl_conformance_v1_0.py
# run a single conformance test
./run_tests.sh -api test/api/test_cwl_conformance_v1_0.py:CwlConformanceTestCase.test_conformance_v1_0_cl_basic_generation
# run the tests that should be succeeding
./run_tests.sh -api test/api/test_cwl_conformance_green_v1_0.py
```

# Creating the tests used by nose
The conformance tests are "mirrored" in the Galaxy repository: the `conformance_tests.yaml` file and its dependencies is in the `test/unit/tools/cwl_tools/v1.0` directory. The script to create the unit tests modules for CWL is `test/unit/tools/cwl_tools/conformance_to_test_cases.py`. When it is run, it parses the conformance_tests file and creates two modules, one for all the CWL conformance tests and one for the ones which are passing, `test/api/test_cwl_conformance_v1_0.py` and `test/api/test_cwl_conformance_green_v1_0.py`
The generation script can be run with:

```shell
python test/unit/tools/cwl_tools/conformance_to_test_cases.py
```

# Install CWL tools in Galaxy

## turn on `enable_beta_tool_formats`
The config file is config/galaxy.yml. If not already done, just copy it from config/galaxy.yml.sample (which is used by default but _do not modify the sample file, copy and modify_.

The part that should appear in there is:

```yaml
  enable_beta_tool_formats: true
```

## add the CWL tools to the tools configuration

modify the list of tools in `config/tool_conf.xml` to add the CWL tools you want to load, e.g.:

```xml
<?xml version='1.0' encoding='utf-8'?>
<toolbox monitor="true">
  <section id="test_cwl" name="CWL test tools">
    <tool file="../test/unit/tools/cwl_tools/galactic_flavored/galactic_cat.cwl" />
  </section>
  <section id="getext" name="Get Data">
    <tool file="data_source/upload.xml" />
  </section>
</toolbox>
```

## start!

. run.sh

# Miscelaneous Client development notes

the "Galaxy inception bug" came from the fact that the form was not detected as "regular", something John corrected in https://github.com/common-workflow-language/galaxy/commit/9291974b7a2c55e37e268cdf9f3b199b87d5a0ca. The code there (`client/galaxy/scripts/mvc/tool/tools.js`) tests for a list of tool types that are accepted as "regular" forms.
When modifying the client code, make sure you rebuild the client with `make client` and reload the portal. To watch the client code (js/css) and rebuild as soon as there is a modification, just use `make client-watch`.
