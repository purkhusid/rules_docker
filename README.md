# Docker Rules

## Rules

* [docker_build](#docker_build)
* [docker_pull](#docker_pull)

## Overview

This repository contains a set of rules for pulling down base images, augmenting
them with build artifacts and assets, and (coming soon) publishing those images.
These rules do not require / use Docker for pulling, building, or pushing
images.  This means:
* They can be used to develop Docker containers on Windows / OSX without
`boot2docker` or `docker-machine` installed.
* They do not require root access on your workstation.

Also unlike traditional Docker builds, the Docker images produced by
`docker_build` are deterministic / reproducible.


## Setup

Add the following to your `WORKSPACE` file to add the external repositories:

```python
git_repository(
    name = "io_bazel_rules_docker",
    remote = "https://github.com/bazelbuild/rules_docker.git",
    # TODO(mattmoor): Update after initial release.
    tag = "0.0.0",
)

load("@io_bazel_rules_docker//docker:docker.bzl", "docker_pull")

docker_pull(
  name = "java_base",
  registry = "gcr.io",
  # TODO(mattmoor): Replace this.
  repository = "google-appengine/java",
  # 'tag' is also supported, but digest is encouraged for reproducibility.
  digest = "sha256:deadbeef",
)
```

## Examples

### docker_build

```python
docker_build(
    name = "app",
    # References docker_pull from WORKSPACE (above)
    base = ["@java_base//image"],
    files = ["//java/com/example/app:Hello_deploy.jar"],
    cmd = ["Hello_deploy.jar"]
)
```

### docker_pull (launcher.gcr.io)

TODO(mattmoor): Add a launcher example.

### docker_pull (DockerHub?)

TODO(mattmoor): Add a DockerHub example.

### docker_pull (Quay?)

TODO(mattmoor): Add a Quay example.


## Things still missing:

- Authenticated pull
- [Authenticated] push



<a name="docker_pull"></a>
## docker_pull

```python
docker_pull(name, registry, repository, digest, tag)
```

A repository rule that pulls down a Docker base image in a manner suitable for
use with `docker_build`s `base` attribute.

<table class="table table-condensed table-bordered table-params">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <thead>
    <tr>
      <th colspan="2">Attributes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <p><code>Name, required</code></p>
        <p>Unique name for this repository rule</p>
      </td>
    </tr>
    <tr>
      <td><code>registry</code></td>
      <td>
        <p><code>Registry Domain; required</code></p>
        <p>The registry from which to pull the base image.</p>
      </td>
    </tr>
    <tr>
      <td><code>repository</code></td>
      <td>
        <p><code>Repository; required</code></p>
        <p>The `repository` of images to pull from.</p>
      </td>
    </tr>
    <tr>
      <td><code>digest</code></td>
      <td>
        <p><code>string; optional</code></p>
        <p>The `digest` of the docker image to pull from the specified
           `repository`.</p>
        <p>
          <strong>Note:</strong> For reproducible builds, use of `digest`
          is recommended.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>tag</code></td>
      <td>
        <p><code>string; optional</code></p>
        <p>The `tag` of the docker image to pull from the specified `repository`.
           If neither this nor `digest` is specified, this attribute defaults
           to `latest`.  If both are specified, then `tag` is ignored.</p>
        <p>
          <strong>Note:</strong> For reproducible builds, use of `digest`
          is recommended.
        </p>
      </td>
    </tr>
  </tbody>
</table>

<a name="docker_build"></a>
## docker_build

```python
docker_build(name, base, data_path, directory, files, legacy_repository_naming, mode, tars, debs, symlinks, entrypoint, cmd, env, labels, ports, volumes, workdir, repository)
```

<table class="table table-condensed table-bordered table-implicit">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <thead>
    <tr>
      <th colspan="2">Implicit output targets</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code><i>name</i>.tar</code></td>
      <td>
        <code>The full Docker image</code>
        <p>
            A full Docker image containing all the layers, identical to
            what <code>docker save</code> would return. This is
            only generated on demand.
        </p>
      </td>
    </tr>
    <tr>
      <td><code><i>name</i>-layer.tar</code></td>
      <td>
        <code>An image of the current layer</code>
        <p>
            A Docker image containing only the layer corresponding to
            that target. It is used for incremental loading of the layer.
        </p>
        <p>
            <b>Note:</b> this target is not suitable for direct consumption.
            It is used for incremental loading and non-docker rules should
            depends on the docker image (<i>name</i>.tar) instead.
        </p>
      </td>
    </tr>
    <tr>
      <td><code><i>name</i></code></td>
      <td>
        <code>Incremental image loader</code>
        <p>
            The incremental image loader. It will load only changed
            layers inside the Docker registry.
        </p>
      </td>
    </tr>
  </tbody>
</table>

<table class="table table-condensed table-bordered table-params">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <thead>
    <tr>
      <th colspan="2">Attributes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <code>Name, required</code>
        <p>A unique name for this rule.</p>
      </td>
    </tr>
    <tr>
      <td><code>base</code></td>
      <td>
        <code>File, optional</code>
        <p>
            The base layers on top of which to overlay this layer, equivalent to
            FROM.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>data_path</code></td>
      <td>
        <code>String, optional</code>
        <p>Root path of the files.</p>
        <p>
          The directory structure from the files is preserved inside the
          docker image but a prefix path determined by `data_path`
          is removed from the directory structure. This path can
          be absolute from the workspace root if starting with a `/` or
          relative to the rule's directory. A relative path may starts with "./"
          (or be ".") but cannot use go up with "..". By default, the
          `data_path` attribute is unused and all files are supposed to have no
          prefix.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>directory</code></td>
      <td>
        <code>String, optional</code>
        <p>Target directory.</p>
        <p>
          The directory in which to expand the specified files, defaulting to '/'.
          Only makes sense accompanying one of files/tars/debs.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>files</code></td>
      <td>
        <code>List of files, optional</code>
        <p>File to add to the layer.</p>
        <p>
          A list of files that should be included in the docker image.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>legacy_repository_naming</code></td>
      <td>
        <code>Bool, default to False</code>
        <p>
          Whether to use the legacy strategy for naming the repository name
          embedded in the resulting tarball.
          e.g. <code>bazel/{target.replace('/', '_')}</code>
          vs. <code>bazel/{target}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td><code>mode</code></td>
      <td>
        <code>String, default to 0555</code>
        <p>
          Set the mode of files added by the <code>files</code> attribute.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>tars</code></td>
      <td>
        <code>List of files, optional</code>
        <p>Tar file to extract in the layer.</p>
        <p>
          A list of tar files whose content should be in the docker image.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>debs</code></td>
      <td>
        <code>List of files, optional</code>
        <p>Debian package to install.</p>
        <p>
          A list of debian packages that will be installed in the docker image.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>symlinks</code></td>
      <td>
        <code>Dictionary, optional</code>
        <p>Symlinks to create in the docker image.</p>
        <p>
          <code>
          symlinks = {
           "/path/to/link": "/path/to/target",
           ...
          },
          </code>
        </p>
      </td>
    </tr>
    <tr>
      <td><code>user</code></td>
      <td>
        <code>String, optional</code>
        <p><a href="https://docs.docker.com/reference/builder/#user">The user
               that the image should run as.</a></p>
        <p>Because building the image never happens inside a docker container,
               this user does not affect the other actions (e.g.,
               adding files).</p>
      </td>
    </tr>
    <tr>
      <td><code>entrypoint</code></td>
      <td>
        <code>String or string list, optional</code>
        <p><a href="https://docs.docker.com/reference/builder/#entrypoint">List
               of entrypoints to add in the image.</a></p>
      </td>
    </tr>
    <tr>
      <td><code>cmd</code></td>
      <td>
        <code>String or string list, optional</code>
        <p><a href="https://docs.docker.com/reference/builder/#cmd">List
               of commands to execute in the image.</a></p>
      </td>
    </tr>
    <tr>
      <td><code>env</code></td>
      <td>
        <code>Dictionary from strings to strings, optional</code>
        <p><a href="https://docs.docker.com/reference/builder/#env">Dictionary
               from environment variable names to their values when running the
               docker image.</a></p>
        <p>
          <code>
          env = {
            "FOO": "bar",
            ...
          },
          </code>
        </p>
      </td>
    </tr>
    <tr>
      <td><code>labels</code></td>
      <td>
        <code>Dictionary from strings to strings, optional</code>
        <p><a href="https://docs.docker.com/reference/builder/#label">Dictionary
               from custom metadata names to their values. You can also put a
               file name prefixed by '@' as a value. Then the value is replaced
               with the contents of the file.
        <p>
          <code>
          labels = {
            "com.example.foo": "bar",
            "com.example.baz": "@metadata.json",
            ...
          },
          </code>
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ports</code></td>
      <td>
        <code>String list, optional</code>
        <p><a href="https://docs.docker.com/reference/builder/#expose">List
               of ports to expose.</a></p>
      </td>
    </tr>
    <tr>
      <td><code>volumes</code></td>
      <td>
        <code>String list, optional</code>
        <p><a href="https://docs.docker.com/reference/builder/#volumes">List
               of volumes to mount.</a></p>
      </td>
    </tr>
    <tr>
      <td><code>workdir</code></td>
      <td>
        <code>String, optional</code>
        <p><a href="https://docs.docker.com/reference/builder/#workdir">Initial
               working directory when running the docker image.</a></p>
        <p>Because building the image never happens inside a docker container,
               this working directory does not affect the other actions (e.g.,
               adding files).</p>
      </td>
    </tr>
    <tr>
      <td><code>repository</code></td>
      <td>
        <code>String, default to `bazel`</code>
        <p>The repository for the default tag for the image.</a></p>
        <p>Image generated by `docker_build` are tagged by default to
           `bazel/package_name:target` for a `docker_build` target at
           `//package/name:target`. Setting this attribute to
           `gcr.io/dummy` would set the default tag to
           `gcr.io/dummy/package_name:target`.</p>
      </td>
    </tr>
  </tbody>
</table>
