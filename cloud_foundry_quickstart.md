# Cloud Foundry quickstart

In brief:

* make sure you have a working BOSH Lite install
* install [Spiff](https://github.com/cloudfoundry-incubator/spiff/releases), a `dynaml` (CF-specific YAML implementation) parser
* clone the [`cf-release`](https://github.com/cloudfoundry/cf-release) repo into the same root dir as BOSH lite
* run `bosh-lite/bin/provision_cf`

## Smoke tests and integration tests

### Running locally
* install go and set GOPATH
* `go get -d github.com/cloudfoundry/cf-acceptance-tests`
* more info: [CF Acceptance Tests](https://github.com/cloudfoundry/cf-acceptance-tests)
* create an `integration_config.yml` per the instructions in the README (including skipping SSL validation as they are hitting your local machine)
* run the tests: `./bin/test`

### Run as BOSH errand
* See BOSH lite quickstart.

## Deploying a sample app
*Assuming a buildpack is present for your chosen language.*

### Prepare your app
* Sinatra apps require a `config.ru`. CF will also use a Procfile if one is present.
* add a `manifest.yml` to the root dir of your app. At the minimum, this must have an `applications` node that contains one `name` entry:

```
---
applications:
- name: hello-env
```

### Prepare CF and push
* point your CF CLI to your CF install. For BOSH Lite running locally, run `cf api api.10.244.0.34.xip.io --skip-ssl-validation`.
* login to CF using `cf login`. For BOSH Lite, login as admin/admin.
* create an org and a space on CF, and target them for deployment:

```
cf create-org <orgName>
cf create-space -o <orgName> <spaceName>
cf target -o <orgName> -s <spaceName>
```

* run `cf push`.

## Scaling an app

* with the app deploying, run `cf scale <app_name> -i N`, where `N` is the number of instances you want.
* `cf apps` 

## Setting app environment variables

* e.g. `cf set-env <app_name> SECRET hushhush`
* `cf env <app_name>` to show all environment variables for an app
* `cf unset-env ...` to remove

## Blue-Green deployment

* We have two releases in production called blue and green, in CF these are separate 'apps'.
* Say the first release is blue on the live route: `cf push blue -n route`
* The next release is green on a temporary route: `cf push green -n route-temp`
* Then the green release can be tested on the temporary route
* To switch over add live route to green: `cf map-route green 10.244.0.34.xip.io -n route`, which starts load balancing between blue and green
* Remove the live route to blue: `cf unmap-route blue 10.244.0.34.xip.io -n route`, so all traffic is routed to green
* Done! The next release will be to blue. Rinse & repeat.

## Application logs

### Loggregator buffer
* Captures CF logging and STDOUT and STDERR from the app
* Tail the loggregator buffer for an app: `cf logs <app_name>`
* Dump the recent logs: `cf logs <app_name> --recent`

### Using a remote logging service
* Like Papertrail, Splunk etc.
* Create a free Papertrail account and a system, use Heroku/Cloud Foundry, copy drain url
* Create a user provided service: `cf cups papertrail -l <syslog_drain_url>`, ensure the url starts with `syslog://`
* Bind the CUPS to an app: `cf bind-service <app_name> papertrail`
* Push or restage the app to start drain: `cf restage <app_name>`

### Inspect log files
* `cf files <app_name> app/<path_to_log_file>` outputs the whole file(!)

## UAA management

### Provisioning a UAA server
* Install Java SDK and follow the quick start and CF deployment guide at https://github.com/cloudfoundry/uaa
* Install the `cf-uaac` gem 
* Target the UAA server: `uaac target https://uaa.10.244.0.34.xip.io --skip-ssl-validation`
* Get the access token from the server: `uaac token client get admin -s <admin_secret>` – the admin secret is set in `bosh-lite/manifests/cf-manifest.yml`; `admin-secret` by default

### Managing users
* get a list of all usernames: `uaac users -a username`
* creating an admin user: 
 
```
uaac user add <user> -p <password> --emails <email>
uaac member add cloud_controller.admin <user>
uaac member add uaa.admin <user>
uaac member add scim.read <user>
uaac member add scim.write <user>
```

## Custom User Provided Services

### Redis Labs
* Create an account and free tier plan, get the host, port and password
* Add a CUPS using JSON, e.g. `cf cups <service_name> -p '{"port":<port>,"host":<host>,"password":<password>}'`
* Or interatively: `cf cups <service_name> -p "port, host, password"`
* Bind the service to an app: `cf bind-service <service_name> <app_name>`
* Restage the app: `cf restage <app_name>`

## Security groups

* these restrict *outbound* connections from apps
* the default running security groups, `public_networks`, is extremely permissive, allowing full access to the public internet

### Adding a security group to an app
* create a new security group: `cf create-security-group <name> <path_to_config_json>`
* apply this security group to your desired spaces: `cf bind-security-group <name> <org> <space>`
* remove this default security group using `cf unbind-running-security-group public_networks`
* after making changes to security groups, restart your apps using `cf restart <app_name>`

## Accessing a container using SSH (when using BOSH Lite)
* find the id of the Warden container you wish to access – there is one per app instance. To do this, run `bosh ssh` and select the DEA (which is labelled as 'runner') to access the CF instance. Run `htop`, then hit F5 to view as tree and sort by `TIME+` to find resource-intensive apps - this will show up the parent Warden container and its id.
* then, to open a shell into the Warden container:

```
sudo su -
cd /var/vcap/data/warden/depot/<container_id>
bin/wsh
```

* from there, use `ps -ef` to show a list of running processes.
* the app itself is located in `/app`.

## Deploying non-web apps in the same repo

* either create a separate manifest.yml for the worker app:
    * create e.g. `manifest.worker.yml`, with a single worker app specified (with `no-route: true` and the appropriate `command` to launch it)
    * deploy app(s) in the separate manifest with `cf push -f <path_to_manifest_file>`
* or put all apps in the same manifest (and specify which to push with `cf push <app_name>`)

## Viewing job spec files

* jobs are tasks that can be run by BOSH. Errands are specific type of job that are run only once, or on demand.
* jobs are found in the `jobs` directory of a release
* each job has a `spec` file which lists its name, description, packages required, templates to be interpreted at runtime, and properties (and their optional defaults and descriptions)
* properties can be set in:
    * job spec (job-specific)
    * manifest job block (job-specific)
    * manifest properties block (global for all jobs)
* properties set in the job spec override all others.
* properties set in the job block are only accessible to that job, and override the global properties set in the properties block.

## Customising CF release

### Changing the manifest
* Edit the config stubs in `cf-release/templates`, e.g. add a security group definition
* Then make the manifest and deploy CF with BOSH.

### Deploying code changes
* fork the submodule repo containing the change (e.g. `cloud_controller_ng`) and the main Cloud Foundry repo
* point `cf-release` to the new submodule repo by modifying the correct entry in `.gitmodules`
* from the `cf-release` directory, run `git submodule sync` to update the remote of the submodule
* go into the submodule directory, make your desired change to the submodule, commit and push
* from the `cf-release` dir, commit the change to the submodule and push

### Creating and deploying a release
* run `bosh create release` to generate the new release - when prompted for a name, hit Enter to use the default ('cf'). By default, this creates a dev release
* run `bosh upload release` to push the new release 
* run `bosh deploy`

If the new release is not the latest release by version number, retarget `bosh deploy` by editing `bosh-lite/manifests/cf-manifest.yml` and changing releases:version (which by default is set to 'latest') to the exact version to be deployed.

## Updating CF release

* Run `cf-release/update` to pull and update all the submodules.
* Deploy CF with BOSH.

## Monitoring NATS message bus traffic

### Find app GUID and NATS config
* Retrieve NATS config from the CF manifest, or from the NATS vm
    - `bosh ssh`
    - Select the NATS vm
    - `ps -ef | grep nats.conf`
    - `more </path/to/nats.conf>`
    - Note the username, password, host and port
* Get the GUID of an app using `cf app <app_name> --guid`

### Monitor traffic for a single app from local machine
* Install the NATS Ruby client: `gem install nats`
* Monitor all NATS traffic: `nats-sub <subject> -s <server>` 
    - `">"` matches any non-blank subject
    - BOSH Lite server is at `nats://nats:nats@10.244.0.6:4222` by default
* Filter traffic by the app GUID: `./nats-all.rb | grep <guid>`

## Hints and tips

* see the full HTTP trace of a command sent to CF by setting `CF_TRACE` to `true`.

