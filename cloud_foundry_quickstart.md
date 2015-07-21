# Cloud Foundry quickstart

In brief:

* make sure you have a working BOSH Lite install
* install (Spiff)[https://github.com/cloudfoundry-incubator/spiff/releases], a `dynaml` (CF-specific YAML implementation) parser
* clone the (`cf-release`)[https://github.com/cloudfoundry/cf-release] repo into the same root dir as BOSH lite
* run `bosh-lite/bin/provision_cf`

## Smoke tests and integration tests
### Running locally
* once CF is deployed, clone the (acceptance tests)[https://github.com/cloudfoundry/cf-acceptance-tests]
* create an `integration_config.yml` per the instructions in the README (including skipping SSL validation as they are hitting your local machine)
* run the tests from `bosh-lite/bin/test`

### Running remotely as errands
* `bosh errands` will list the available jobs (including test suites)
* `bosh run errand <name>` will run an errand on the VM