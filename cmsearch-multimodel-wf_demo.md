# Running *cmsearch-multimodel-wf* in Galaxy-CWL - annex 3

## *cmsearch-multimodel-wf* demo procedure

* Workflow import

    * Set tools path in workflow file according to your environment.

          cd ~/snapshot/workflow-is-cwl
          sed -i 's/foobar/vagrant/' workflows/cmsearch-multimodel-wf.cwl # replace 'vagrant' with your user

    * Open url below

          http://localhost:8080

    * Logon as demo@g-cwl-is.test

    * Click on "Workflow"

    * Click on "Import workflow" (vertical arrow)

    * Click on "Choose file" and select the workflow (cmsearch-multimodel-wf.cwl)

    * Click on "Import Workflow"

      Note: the message "Import failed for unkown reason" may appear
      but this doesn't disrupt the import process (the workflow is still
      imported. Maybe this is because the import process takes too long.
      FIXME.

* Workflow run

    * Load input data in Galaxy

        * mt-tmRNA.cm
        * tRNA5.c.cm
        * mrum-genome.fa
        * ribo.claninfo

    *  Merge *.cm files into a collection

        * Select files below and click on "Build a dataset list"

            * mt-tmRNA.cm
            * tRNA5.c.cm

        * Check "Hide original element"

        * Set list name to "cmlist"

    * Click on "Workflow"

    * In workflow action menu, click on "Run"

    * In the "Run workflow" form, select the correct input

        * cmlist
        * mrum-genome.fa
        * ribo.claninfo

    * In "cmsearch" step parameter, set value below

        * omit_alignment_section: true
        * only_hmm: true
        * search_space_size: 1000

    * Click on "Run Workflow"
