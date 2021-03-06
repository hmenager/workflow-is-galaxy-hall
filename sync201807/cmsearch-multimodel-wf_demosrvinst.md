# Running *cmsearch-multimodel-wf* in Galaxy-CWL - annex&nbsp;2 - demo server installation procedure

* Install a Linux box (eg Ubuntu 16)

* Install deps

    * Required
        * ```apt-get install git python-dev gcc make```
    * Optional
        * ```apt-get install screen sqlite3 cdargs netcat-traditional vim nginx apache2-utils```

* Install docker

    * ```apt-get install docker.io```
    * Add galaxy user in docker group (in /etc/group)
    * Log off/log on (to reset user groups)

* Install galaxy

      cd $HOME/snapshot
      git clone https://github.com/hmenager/galaxy.git
      cp database/demo.sqlite database/universe.sqlite

  Some Galaxy configuration files have been set to fit the demo server needs:

    * config/job_conf.xml

      This configuration file have been set to allow running CWL tools in Docker container.

          <?xml version="1.0"?>
          <job_conf>
              <plugins>
                  <plugin id="local" type="runner" load="galaxy.jobs.runners.local:LocalJobRunner" workers="4"/>
              </plugins>
              <destinations default="docker_local">
                  <destination id="local" runner="local"/>
                  <destination id="docker_local" runner="local">
                      <param id="docker_enabled">true</param>
                      <param id="docker_sudo">false</param>
                  </destination>
              </destinations>
          </job_conf>

    * config/galaxy.yml

        * allow_path_paste: true
        * enable_beta_tool_formats: true
        * admin_users: demo@g-cwl-is.test

    * config/tool_conf.xml

      This configuration file have been set to load CWL tools at Galaxy startup.

          <section id="test_cwl" name="CWL test tools">
              <tool file="../../workflow-is-cwl/tools/Diamond/Diamon.makedb-v0.9.21.cwl" />
              <tool file="../../workflow-is-cwl/tools/TransDecoder/TransDecoder.LongOrfs-v5.cwl" />
              <tool file="../../workflow-is-cwl/tools/Infernal/cmsearch/infernal-cmsearch-v1.1.2.cwl" />
              <tool file="../../workflow-is-cwl/tools/InterProScan/InterProScan-v5.cwl" />
              <tool file="../../workflow-is-cwl/tools/HMMER/phmmer-v3.2.cwl" />
              <tool file="../../workflow-is-cwl/tools/BUSCO/BUSCO-v3.cwl" />
              <tool file="../../workflow-is-cwl/tools/cmsearch-deoverlap/cmsearch-deoverlap-v0.02.cwl" />
          </section>

* Install test tools and workflows

      cd ~/snapshot
      git clone https://github.com/hmenager/workflow-is-cwl

* Start / stop galaxy (to create .venv folder)

      cd ~/snapshot/galaxy
      sh run.sh --daemon
      sh run.sh --stop-daemon

* Install cwltool

      cd ~/snapshot
      git clone https://github.com/hmenager/cwltool
      cd cwltool
      git checkout ebi-workflow-is # demo branch
      source ../galaxy/.venv/bin/activate
      pip uninstall -y cwltool && pip install .

* Copy demo database

  This database contains 'demo' user

    * Email: demo@g-cwl-is.test
    * Public name: demo

   ```cp database/demo.sqlite database/universe.sqlite```

* Start galaxy

      sh run.sh --daemon

* Test

  Demo procedure is detailed here:

  <https://github.com/hmenager/workflow-is-galaxy-hall/blob/master/cmsearch-multimodel-wf_demo.md>

* Stop galaxy

      sh run.sh --stop-daemon
