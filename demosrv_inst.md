# CWL-IS demo server installation procedure

* Install a Linux box (eg Ubuntu 16)

* Install deps
    * Required
        * ```apt-get install git python-dev gcc make docker.io```
    * Optional
        * ```apt-get install screen sqlite3 cdargs netcat-traditional vim nginx apache2-utils```

* Install docker

    * Add galaxy user in docker group (in /etc/group)
    * Log off/log on (to reset user groups)

* Install galaxy

    * ```cd $HOME/snapshot
      git clone https://github.com/hmenager/galaxy.git
      cp database/demo.sqlite database/universe.sqlite```

* Install test tools and workflows

    * cd ~/snapshot
    * git clone https://github.com/hmenager/workflow-is-cwl

* Set tools path in workflows files according to your configuration

    * cd ~/snapshot/workflow-is-cwl
    * sed -i 's/foobar/vagrant/' workflows/cmsearch-multimodel-wf.cwl # replace 'vagrant' with your user

* Install cwltool

    * cd ~/snapshot
    * git clone https://github.com/hmenager/cwltool
    * cd cwltool
    * git checkout ebi-workflow-is # demo branch
    * source ../galaxy/.venv/bin/activate
    * pip uninstall -y cwltool && pip install .

* Start galaxy

    * sh run.sh --daemon

* Test

    * http://localhost:8080

* Stop galaxy

    * sh run.sh --stop-daemon
