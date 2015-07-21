# Cloud Foundry quickstart

In brief:

* make sure you have a working BOSH Lite install
* install [Spiff](https://github.com/cloudfoundry-incubator/spiff/releases), a `dynaml` (CF-specific YAML implementation) parser
* clone the [`cf-release`](https://github.com/cloudfoundry/cf-release) repo into the same root dir as BOSH lite
* run `bosh-lite/bin/provision_cf`

## Smoke tests and integration tests
### Running locally
* once CF is deployed, clone the [acceptance tests](https://github.com/cloudfoundry/cf-acceptance-tests)
* create an `integration_config.yml` per the instructions in the README (including skipping SSL validation as they are hitting your local machine)
* run the tests from `bosh-lite/bin/test`

### Running remotely as errands
* `bosh errands` will list the available jobs (including test suites)
* `bosh run errand <name>` will run an errand on the VM


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

## Hints and tips

* see the full HTTP trace of a command sent to CF by setting `CF_TRACE` to `true`.