# Blue/Green deployer plugin for CF

<div class="warning">

**WARNING**
This repository has no more sense to be developed. It is kept only for the historical reasons.
Cloud Foundry CLI client starting from v7.0 supports rolling deployment strategy out of the box.
</div>

## Introduction

**cf-blue-green-deploy** is a plugin for the CF command line tool that
automates a few steps involved in zero-downtime deploys.

## Overview

The plugin takes care of the following steps packaged into one command:

* Pushes the current version of the app with a new name
* Optionally runs smoke tests against the newly pushed app to verify the deployment
  * If smoke tests fail, newly pushed app gets marked as failed and left around for investigation
  * If smoke tests pass, remaps routes from the currently live app to the newly deployed app
* Cleans up versions of the app no longer in use

## How to install

### Download and install pre-built version

The shell fragment below should work for appropriate version and Linux platform please tune it to meet your conditions.

```
CF_BGD_VERSION="1.5.1a"
CF_BGD_PLATFORM="linux64"
curl -L -o "/tmp/blue-green-deploy" \
  "https://github.com/romanik/cf-blue-green-deploy/releases/download/v${CF_BGD_VERSION}/blue-green-deploy.${CF_BGD_PLATFORM}" \
  && chmod +x "/tmp/blue-green-deploy" \
  && cf install-plugin -f "/tmp/blue-green-deploy" \
  && rm "/tmp/blue-green-deploy"
```

### Install non-forked original plugin from the CF Community Repository

Or you can use non-forked original plugin without some improvements introduced here. But generaly it does the job.

```
cf add-plugin-repo CF-Community https://plugins.cloudfoundry.org
cf install-plugin blue-green-deploy -r CF-Community
```

### If nothing suits

Look down for the instructions how to build the plugin yourself.

## How to use

* Deploy your app

```
cd your_app_root
cf blue-green-deploy app_name
```

* Deploy with optional smoke tests

```
cf blue-green-deploy app_name --smoke-test <path to test script>
```

* Deploy with specific manifest file

```
cf blue-green-deploy app_name -f <path to manifest>
```

* Deploy with a hard clean-up of the 'blue' (original) app

```
cf blue-green-deploy app_name --delete-old-apps
```

* You can also use the shorter alias

```
cf bgd app_name
```

The only argument passed to the smoke test script is the FQDN of the newly
pushed app. If the smoke test returns with a non-zero exit code the deploy
process will stop and fail, the current live app will not be affected.

If the test script exits with a zero exit code, the plugin will remap all
routes from the current live app to the new app. The plugin supports routes
under custom domains.

## How to build

Before cloning the source, you may wish to set up GOPATH and a go-friendly folder hierarchy to avoid path issues. Run the following in your preferred working directory:

```
mkdir ./go
export GOPATH=`pwd`/go
mkdir -p go/src/github.com/romanik/
cd go/src/github.com/romanik/
git clone https://github.com/romanik/cf-blue-green-deploy
cd cf-blue-green-deploy
```

Then run a build:

```
script/build
```

This will download dependencies, run the tests, and build binaries in the
_artefacts_ folder.

## How to run tests

```
script/test
```

This will run the unit tests. To run the acceptance tests (which need a Cloud Foundry instance), use

```
script/test_acceptance
```

You almost certainly want to install the plugin before running the acceptance tests (to make sure the latest version of the plugin is being tested). On OS X, the command would be

```
script/build ; script/install ; script/test_acceptance
```

See [instructions for releasing a project](https://github.com/romanik/cf-blue-green-deploy/blob/master/release.md)
for instructions on how to setup the acceptance tests.

```

```
