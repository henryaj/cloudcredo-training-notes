# Concourse quickstart

## Deploying sample 'Hello World' app
### Deploying Concourse

* follow the instructions at http://concourse.ci/deploying-and-upgrading-concourse.html to upload the Concourse and Garden Linux release to your BOSH install:

```
bosh upload release https://github.com/concourse/concourse/releases/download/v0.54.0/concourse-0.54.0.tgz
bosh upload release https://github.com/concourse/concourse/releases/download/v0.54.0/garden-linux-0.205.0.tgz
```

* use the included Concourse BOSH Lite sample [manifest file](https://github.com/concourse/concourse/blob/master/manifests/bosh-lite.yml) and deploy the release:

`bosh deployment <path_to_manifest>`

* add a local route to the new Concourse deployment:

`sudo route add -net 10.244.8.0/30 192.168.50.4`

* visit 10.244.8.0.xip.io:8080 to see your empty Concourse pipeline!

### Installing fly and creating a pipeline
* download `fly` from the Concourse web interface and make sure the executable is in your $PATH.
* configure a target using the `--target` flag, giving it a name:
`fly save-target --api http://10.244.8.2:8080 <name>`
* create a build plan - in this case, the example plan:

```
jobs:
- name: hello-world
  plan:
  - task: say-hello
    config:
      platform: linux
      image: "docker:///busybox"
      run:
        path: echo
        args: ["Hello, world!"]
```

* tell `fly` to configure that sample pipeline:

`fly --target <name> configure sample-pipeline -c sample-build-plan.yml`

* now view the pipeline at 10.244.8.0.xip.io:8080. The task will be paused by default, and can be started from the web interface or on the command line (using `fly --target <name> configure sample-pipeline --paused=false`)

## Running one-off test jobs
* Use `fly execute` to run a task.
* The task config must be in a separate YAML file, e.g. `fly --target <target_name> execute -c <task.yml>`.
* Use the option `-p` to execute the task with root privileges.

## Keeping `fly` up-to-date
* update `fly` to the correct version for your Concourse deployment:
`fly --target <name> sync`
* targeting a Concourse deployment is a convenience feature for storing your credentials in a .flyrc file for later use