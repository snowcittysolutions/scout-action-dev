# About

GitHub Action to run the Docker Scout CLI as part of your workflows.

You can pick one of the following commands to run:

- `quickview`: get a quick overview of an image, base image and available recommendations
- `compare`: compare an image to a second one (for instance to `latest`)
- `cves`: display vulnerabilities of an image
- `recommendations`: display available base image updates and remediation recommendations
- `sbom`: generate the SBOM of the image
- `environment`: record an image to an environment

[![Screenshot](.github/compare_pr_comment.png)](https://github.com/docker/scout-demo-service/pull/2)

# Inputs

## Command

You can run one or multiple commands in the same GitHub Action run. Use a comma separated list to run several commands.

| <!-- -->   | <!-- -->     | <!-- --> | <!-- -->                                                                                                                                                                                                                          |
|:-----------|:-------------|:---------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `command`  | **required** | `string` | Single command to run or comma separated list of commands to run in order.<br/>Possible values:<br/><ul><li>`quickview`</li><li>`compare`</li><li>`cves`</li><li>`recommendations`</li><li>`sbom`</li><li>`environment`</li></ul> |

The commands will be run in the order of the value, and will share the same parameters.

For instance, if you built an image and want to display a `quickview` as well as to `compare` it against the latest indexed one, set the action as following:

```yaml
command: quickview,compare
image: ${{ steps.meta.outputs.tags }}
to-latest: true
```

## Authentication

### Login to Docker Hub

To use `scout` features you need to be authenticated against Docker Hub.

You can use the parameters below to authenticate, or you can use the [`docker/login-action`](https://github.com/docker/login-action). 

| <!-- -->             | <!-- -->     | <!-- --> | <!-- -->                   |
|:---------------------|:-------------|:---------|:---------------------------|
| `dockerhub-user`     | **optional** | `string` | Docker Hub user id         |
| `dockerhub-password` | **optional** | `string` | Docker Hub password or PAT |


### Login to a registry to pull private images (non Docker Hub)


| <!-- -->            | <!-- -->                                         | <!-- --> | <!-- -->                                |
|:--------------------|:-------------------------------------------------|:---------|:----------------------------------------|
| `registry-user`     | **required** to pull from other private registry | `string` | Registry user id to pull images         |
| `registry-password` | **required** to pull from other private registry | `string` | Registry password or PAT to pull images |

## Common Inputs

### Image

| <!-- -->   | <!-- -->                                 | <!-- --> | <!-- -->                                                                                |
|:-----------|:-----------------------------------------|:---------|:----------------------------------------------------------------------------------------|
| `image`    | **optional** (*)                         | `string` | Prefixed name of the image, directory or archive to operate on                          |
| `platform` | **optional** current platform by default | `string` | Platform of the image to analyze (or the current platform)                              |
| `ref`      | **optional** default is empty            | `string` | Reference to use if the provided tarball containers multiple images, only with archives |

(*) If `image` is not set (or empty) the most recently built image, if any, will be used instead.

### Prefix

The `image` field can be prefixed to indicate the type of content to analyse.
If no prefix is set, the `image://` prefix is used.

Available prefixes:

- `image://`: optional prefix, the target is an image that first will be resolved locally then, if not found, will be resolved to the associated registry
- `local://`: only resolve the image from the local image store
- `registry://`: do not use the local image store, only use the registry
- `oci-dir://`: local directory to be read as an OCI directory
- `archive://`: image archive generated by `docker save` command, as a `tar` file
- `fs://`: local directory or file

### Organization namespace

Namespace of the Docker Organization is required to match the query with the right data.

When using _environments_ (for instance to compare an image to the one from a defined environment, or when comparing to the latest indexed) `organization` parameter is required.

| <!-- -->       | <!-- -->                                                                                                                                      | <!-- --> | <!-- -->                             |
|:---------------|:----------------------------------------------------------------------------------------------------------------------------------------------|:---------|:-------------------------------------|
| `organization` | **required** to compare to environments/latest indexed<br/>**required** to manage environments<br/>**optional** in other cases, default empty | `string` | Namespace of the Docker organization |

## Step Summary

By default the Markdown output of the command (if supported) will be displayed as a [Job Summary](https://github.blog/2022-05-09-supercharging-github-actions-with-job-summaries/).
This can be disabled if needed.

| <!-- -->  | <!-- -->                       | <!-- -->  | <!-- -->                      |
|:----------|:-------------------------------|:----------|:------------------------------|
| `summary` | **optional** default is `true` | `boolean` | Display output as Job Summary |

## Pull Request Comments

When triggered by a `pull_request` event, the output of the scout command can be written as a comment.

This behaviour is **enabled** by default.

By default one single comment per job step will be kept and updated at each run.
If you prefer to keep previous comments but hide them, set the `keep-previous-comments` parameter to `true`.

`pull-requests: write` permission is required to allow the GitHub action to create the comment.

| <!-- -->                 | <!-- -->                               | <!-- -->  | <!-- -->                                                                                       |
|:-------------------------|:---------------------------------------|:----------|:-----------------------------------------------------------------------------------------------|
| `github-token`           | **optional** default is `github.token` | `string`  | GitHub Token to create the comment                                                             |
| `write-comment`          | **optional** default is `true`         | `boolean` | Boolean, write a comment with scout output                                                     |
| `keep-previous-comments` | **optional** default is `false`        | `boolean` | If set, keep but hide previous comment. If not set, keep and update one single comment per job |

## Output

The text version of the command output will be displayed in the logs. The markdown version (if exists) of the command output will be
set as an output of the step, using the command name as identifier, and will be displayed as Pull Request comment or Step Summary.

## `compare` Inputs

### Compare to an image

| <!-- -->  | <!-- -->                        | <!-- --> | <!-- -->                                                                                |
|:----------|:--------------------------------|:---------|:----------------------------------------------------------------------------------------|
| `to`      | **required**                    | `string` | Prefixed name of the image, directory or archive to compare with                        |
| `to-ref`  | **optional** default is empty   | `string` | Reference to use if the provided tarball containers multiple images, only with archives |

See [Prefix](#prefix) above about the available prefixes for the `to` argument.

### Compare to an environment

| <!-- -->    | <!-- -->           | <!-- -->  | <!-- -->                                |
|:------------|:-------------------|:----------|:----------------------------------------|
| `to-env`    | (*)                | `string`  | Name of the environment to compare with |
| `to-stream` | **deprecated** (*) | `string`  | Name of the stream to compare with      |
| `to-latest` | (*)                | `boolean` | Compare to latest indexed image         |

(*) One and only one needs to be defined.


### Common Inputs

| <!-- -->             | <!-- -->                                       | <!-- -->  | <!-- -->                                                                                                  |
|:---------------------|:-----------------------------------------------|:----------|:----------------------------------------------------------------------------------------------------------|
| `ignore-unchanged`   | **optional** default is `false`                | `boolean` | Filter out unchanged packages                                                                             |
| `only-severities`    | **optional** default is empty (all severities) | `string`  | Comma separated list of severities (`critical`, `high`, `medium`, `low`, `unspecified`) to filter CVEs by |
| `only-package-types` | **optional** default is empty (all types)      | `string`  | Comma separated list of package types (like `apk`, `deb`, `rpm`, `npm`, `pypi`, `golang`, etc)            |
| `only-fixed`         | **optional** default is `false`                | `boolean` | Filter to fixable CVEs                                                                                    |
| `only-unfixed`       | **optional** default is `false`                | `boolean` | Filter to unfixed CVEs                                                                                    |
| `exit-code`          | **optional** default is `false`                | `boolean` | Return exit code `2` if vulnerability changes are detected                                                |

## `cves` Inputs

| <!-- -->             | <!-- -->                                       | <!-- -->  | <!-- -->                                                                                                  |
|:---------------------|:-----------------------------------------------|:----------|:----------------------------------------------------------------------------------------------------------|
| `only-severities`    | **optional** default is empty (all severities) | `string`  | Comma separated list of severities (`critical`, `high`, `medium`, `low`, `unspecified`) to filter CVEs by |
| `only-package-types` | **optional** default is empty (all types)      | `string`  | Comma separated list of package types (like `apk`, `deb`, `rpm`, `npm`, `pypi`, `golang`, etc)            |
| `only-fixed`         | **optional** default is `false`                | `boolean` | Filter to fixable CVEs                                                                                    |
| `only-unfixed`       | **optional** default is `false`                | `boolean` | Filter to unfixed CVEs                                                                                    |
| `ignore-base`        | **optional** default is `false`                | `boolean` | Ignore base image vulnerabilities                                                                         |
| `sarif-file`         | **optional** default is empty (no output file) | `string`  | Write output to a SARIF file for further processing or upload into GitHub code scanning                   |

## `recommendations` Inputs

| <!-- -->       | <!-- -->                         | <!-- -->  | <!-- -->                                        |
|:---------------|:---------------------------------|:----------|:------------------------------------------------|
| `only-refresh` | **optional** default is `false`  | `boolean` | Only display base image refresh recommendations |
| `only-update`  | **optional** default is `false`  | `boolean` | Only display base image update recommendations  |

## `environment` Inputs

The `image` input must be an image in your local image store or in a registry.
You can use [prefixes](#prefixes) to control whether to use a local or remote image.
The following prefixes are supported:

- `image://` (optional)
- `local://`
- `registry://`

| <!-- -->      | <!-- -->     | <!-- -->  | <!-- -->                                    |
|:--------------|:-------------|:----------|:--------------------------------------------|
| `environment` | **required** | `string`  | Name of the environment to record the image |

[See Environment example](#record-an-image-deployed-to-an-environment)

## Example usage

### Build an image, push and compare

```yaml
name: Docker

on:
  push:
    tags: [ "*" ]
    branches:
      - 'main'
  pull_request:
    branches: [ "**" ]
    
env:
  # Use docker.io for Docker Hub if empty
  REGISTRY: docker.io
  IMAGE_NAME: ${{ github.repository }}
  SHA: ${{ github.event.pull_request.head.sha || github.event.after }}
  # Use `latest` as the tag to compare to if empty, assuming that it's already pushed
  COMPARE_TAG: latest

jobs:
  build:

    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      pull-requests: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
        with:
          ref: ${{ env.SHA }}
          
      - name: Setup Docker buildx
        uses: docker/setup-buildx-action@v2.5.0
        with:
          driver-opts: |
            image=moby/buildkit:v0.10.6

      # Login against a Docker registry except on PR
      # https://github.com/docker/login-action
      - name: Log into registry ${{ env.REGISTRY }}
        uses: docker/login-action@v2.1.0
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ secrets.DOCKER_USER }}
          password: ${{ secrets.DOCKER_PAT }}

      # Extract metadata (tags, labels) for Docker
      # https://github.com/docker/metadata-action
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v4.4.0
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          labels: |
            org.opencontainers.image.revision=${{ env.SHA }}
          tags: |
            type=edge,branch=$repo.default_branch
            type=semver,pattern=v{{version}}
            type=sha,prefix=,suffix=,format=short
      
      # Build and push Docker image with Buildx (don't push on PR)
      # https://github.com/docker/build-push-action
      - name: Build and push Docker image
        id: build-and-push
        uses: docker/build-push-action@v4.0.0
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Docker Scout
        id: docker-scout
        if: ${{ github.event_name == 'pull_request' }}
        uses: docker/scout-action@v1
        with:
          command: compare
          image: ${{ steps.meta.outputs.tags }}
          to: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ env.COMPARE_TAG }}
          ignore-unchanged: true
          only-severities: critical,high
          write-comment: true
          github-token: ${{ secrets.GITHUB_TOKEN }} # to be able to write the comment
```  

### All-in-one

For the latest built image, display:
- the vulnerabilities (ignoring the base image, only displaying vulnerabilities with a fix available)
- the available recommendations
- compare it to the latest image indexed for the same repository (only displaying unchanged packages and vulnerabilities that already have a fix)

```yaml
        - name: Docker Scout
          id: docker-scout
          uses: docker/scout-action@v1
          with:
            command: cves,recommendations,compare
            to-latest: true
            ignore-base: true
            ignore-unchanged: true
            only-fixed: true
```

### Analyze vulnerabilities and upload report to GitHub code scanning

When GitHub code scanning is enabled, the `sarif-file` input can be used to upload the vulnerabilities to GitHub.

```yaml
      - name: Analyze for critical and high CVEs
        id: docker-scout-cves
        if: ${{ github.event_name != 'pull_request_target' }}
        uses: docker/scout-action@v1
        with:
          command: cves
          image: ${{ steps.meta.outputs.tags }}
          sarif-file: sarif.output.json
          summary: true
      
      - name: Upload SARIF result
        id: upload-sarif
        if: ${{ github.event_name != 'pull_request_target' }}
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: sarif.output.json
```

### Record an image deployed to an environment

```yaml
      - name: Build and push Docker image
        id: build-and-push
        uses: docker/build-push-action@v4.0.0
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Docker Scout
        id: docker-scout-environment
        uses: docker/scout-action@v1
        with:
          command: environment
          image: ${{ steps.meta.outputs.tags }}
          environment: prod
          organization: my-docker-org
```

# License

The Docker Scout CLI is licensed under the Terms and Conditions of the [Docker Subscription Service Agreement](https://www.docker.com/legal/docker-subscription-service-agreement/).
