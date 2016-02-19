# docker-logstash-smoke-tests BOSH release
This BOSH release invokes the smoke tests for Logstash on CF.
The actual smoke tests reside at: http://github.com/starkandwayne/cf-logstash-smoke-tests

## License

MIT, see LICENSE file.

## Usage: Configuration & Delpoyment

We will walk through an example of using with Cloud Foundry.

Be sure to first target your BOSH Director:
```sh
bosh target $BOSH_HOST
```

If you are using bosh-lite you target like so:
```sh
bosh target 192.168.50.4 lite
```

Now clone this release and cd into the directory:
```sh
git clone https://.../docker-logstash-smoke-tests-boshrelease.git
cd docker-logstash-smoke-tests-boshrelease
```

If you intend on using a final release upload it like so:
```sh
bosh upload release releases/docker-logstash-smoke-tests-1.yml
```

Next download your manifest file for the deployment targeted so we can edit it and add the release.

```sh
mkdir -p ~/workspace/manifests
bosh download manifest docker-logstash-smoke-tests-development ~/workspace/manifests/docker-logstash-smoke-tests.yml
bosh deployment ~/workspace/manifests/docker-logstash-smoke-tests.yml
```

Alternatively, you can make your manifest. For example to prepare a manifest for
bosh-lite (warden) using the 'centos' stemcell we would do the following:

```sh
STEMCELL_OS=centos ./docker-logstash-smoke-tests-dev manifest warden
```

Edit the manifest file you downloaded (`~/workspace/manifests/docker-logstash-smoke-tests.yml`) and add settings as follows.

Add to the list of known `releases: `

```yaml
releases:
#...
- name: docker-logstash-smoke-tests
  version: latest
```

For consistency also add to the `releases: ` section under `meta: `

```yaml
meta:
  environment: docker-logstash-smoke-tests-development
  releases:
  - name: cf
    version: latest
  - name: docker-logstash-smoke-tests
    version: latest
```

Add properties such as tags.

```yaml
properties:
# ... lots of properties ... at bottom put vv
  docker-logstash-smoke-tests:
```

Now, for every `instances: ` entry you wish to collocate this release with under `jobs:` add the following in the `templates: ` section:

```yaml
  - name: docker-logstash-smoke-tests
    release: docker-logstash-smoke-tests
```

Now you can deploy,

```yaml
bosh -n deploy
```

Note that for each job you create in your release that you want to run on a
Job VM you must add a `templates:` entry with the `name:` of the template
and the `release:` from which it comes.


You can run the smoke tests with

```
bosh run errand logstash-smoke-tests
```

If you want to keep the vm alive after running the errand

```
bosh run errand logstash-smoke-tests --keep-alive
```

## Deployments Blobs

The script `./docker-logstash-smoke-tests-dev blobs` is used to prepare the `blobs/` directory
runs each package's `prepare` script from within the `blobs/`
directory. This will run each package's `prepare` script, if it exists,
which *should* download and prepare that package's required blobs
(source tarballs, etc...) into the {package}/ directory.

## Development

Download your manifest file for the deployment targeted.

Target your BOSH Director as explained above, if you already have a deploy be sure to also download your manifest as per above.

If this is your first time cloning the release repository for develompent first prepare all of the things:
```sh
./docker-logstash-smoke-tests-dev prepare warden
```

This is equivalent to:
```sh
./docker-logstash-smoke-tests-dev blobs
./docker-logstash-smoke-tests-dev release
./docker-logstash-smoke-tests-dev stemcell warden
./docker-logstash-smoke-tests-dev manifest warden
```

Once these steps have all been completed

```sh
bosh -n deploy
```

## Debugging & QA

See `docs/notes.md` for more information.

In order to gain access to one of the VMs, from a terminal `bosh ssh docker-logstash-smoke-tests {VM Index}`,
for example to ssh to the first node:
```sh
bosh ssh docker-logstash-smoke-tests 0 # Hop on and have a look around...
```

If you want to destroy and recreate on specific node, say `docker-logstash-smoke-tests/0`, you do the following:

```sh
bosh -n recreate docker-logstash-smoke-tests 0 --force
```

See `docs/docker-logstash-smoke-tests.md` for release specific information.
