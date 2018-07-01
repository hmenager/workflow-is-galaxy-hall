# Intro

# Setup

The repository where the Galaxy-CWL work is done is [fork](https://github.com/common-workflow-language/galaxy). We (@hmenager, @khhillion, @jra001k) are working on a [sub-fork of it](https://github.com/common-workflow-language/galaxy).

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
./run_tests.sh -api test/api/test_cwl_conformance_v1_0.py:CwlConformanceTestCase.test_conformance_v1_0_6
```


