# BOSH Lite quickstart

In brief:

* install Vagrant and VMware
* install the BOSH CLI 
* clone (BOSH Lite)[https://github.com/cloudfoundry/bosh-lite] repo
* install and run BOSH Lite image
* use `bosh target` to point BOSH CLI to the local VM

## Stop and go
* starting the VM: `vagrant up`
* suspending the VM: `vagrant suspend`
* killing the VM: `vagrant halt`. After this command your Warden VM will be lost - and will need to be restored with `bosh cck`

## Viewing task logs
* `bosh tasks` returns the log of the last task run.
* `bosh tasks recent <num || 30>` returns a list of recent tasks, excluding certain types of task (like VM repairs). These can be shown with the `--no-filter` flag.
* `bosh task <num>` shows the CLI output shown when a task has run. Add the `--debug` flag to show the full debug-level logging from the remote machine.

## Hints and tips

* sometimes `xip.io` doesn't work -- this is likely the culprit if you get 'hostname not found' errors
* if it's broken, repair your BOSH VM with `bosh cck` ('cloud check')
