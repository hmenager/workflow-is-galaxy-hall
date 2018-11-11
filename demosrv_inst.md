# CWL-IS demo server installation procedure

* Install Ubuntu 16

* Install deps

    * ```aptitude install git python-dev gcc make screen sqlite3 cdargs netcat-traditional vim nginx apache2-utils docker.io```

* Install docker

    * Add galaxy user in docker group (in /etc/group)
    * Log off/log on (to reset user group)

* Install galaxy

    * cd $HOME/snapshot
    * git clone https://github.com/hmenager/galaxy.git
    * cp database/demo.sqlite database/universe.sqlite

* Install test tools and workflows

    * cd ~/snapshot
    * git clone https://github.com/hmenager/workflow-is-cwl

* set tools path in workflows files according to your configuration

    * cd ~/snapshot/workflow-is-cwl
    * sed -i 's/foobar/vagrant/' workflows/cmsearch-multimodel-wf.cwl # replace 'vagrant' with your user

* install cwltool

    * cd ~/snapshot
    * git clone https://github.com/hmenager/cwltool
    * cd cwltool
    * git checkout ebi-workflow-is # demo branch
    * source ../galaxy/.venv/bin/activate
    * pip uninstall -y cwltool && pip install .

* start galaxy

    * sh run.sh --daemon

* test

    * http://sd-104052.dedibox.fr:8083

* stop galaxy

    * sh run.sh --stop-daemon
