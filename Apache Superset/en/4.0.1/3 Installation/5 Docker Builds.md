[TOC]

The Apache Superset community extensively uses Docker for development, release,
and productionizing Superset. This page details our Docker builds and tag naming
schemes to help users navigate our offerings.

Images are built and pushed to the [Superset Docker Hub repository](
https://hub.docker.com/r/apache/superset). Different sets of images are created for:

- **Published releases** (`release`): with tags like `3.0.0` and the `latest` tag.
  Those are multi-platform (arm+amd). More on that later.
- **Pull request iterations** (`pull_request`):, each identified by tags starting with a SHA like
  `8a2f7d378ab13c156fa183d9284b607ed69f5ecc`, and `pr-3454`, referencing the pull
  request ID.
- **Merges to the main branch** (`push`): resulting in new SHAs, with tags
  prefixed with `master` for the latest `master` version.

Each CI build run has multiple builds for different purposes, identified by suffixes:
- **Build preset:** We offer various images for different needs:
    - `lean`: The default Docker image, including both frontend and backend. Tags
      without a build_preset are lean builds, e.g., `latest`.
    - `dev`: For development, with a headless browser and root access.
    - `py310`, e.g., Py310: Similar to lean but with a different Python version (in this example, 3.10).
    - `ci`: For certain CI workloads.
    - `websocket`: For Superset clusters supporting advanced features.
    - `dockerize`: Used by Helm.
- **Platform:** We build for `linux/arm64` and `linux/amd64`. The `-arm` suffix
  indicates ARM builds (e.g., `latest-arm`), while tags without a suffix are for
  AMD (e.g., `latest`).

## Key Image Tags and Examples

- `latest`: The latest official release build, implicitly the lean build on
  `linux/amd64`.
- `latest-dev`: the `-dev` image of the latest official release build, with a
  headless browser and root access.
- `master`: The latest build from the `master` branch, implicitly lean on
  `linux/amd64`.
- `master-dev`: Similar to `master` but includes a headless browser and root access.
- `pr-5252`: The latest commit in PR 5252.
- `30948dc401b40982cb7c0dbf6ebbe443b2748c1b-dev-arm`: A `linux/arm64` build for
  this specific SHA, which could be from a pull request, master merge, or release.
- `30948dc-dev-arm`: Same as above, but SHA truncated to 7 characters for a
  shorter handle on the same image
- `websocket-latest`: The WebSocket image for use in a Superset cluster.

For insights or modifications to the build matrix and tagging conventions,
check the [build_docker.py](https://github.com/apache/superset/blob/master/scripts/build_docker.py)
script and the [docker.yml](https://github.com/apache/superset/blob/master/.github/workflows/docker.yml)
GitHub action.

## Caching

To accelerate builds, we follow Docker best practices and use `apache/superset-cache`.

## About database drivers

Our docker images come with little to zero database driver support since
each envrionment requires different drivers, and mataining a build with
wide database support would be both challenging (dozens of databases,
python drivers, and os dependencies) and inefficient (longer
build times, larger images, lower layer cache hit rate, ...).

For production use cases, we recommend that you derive our `lean` image(s) and
add database support for the database you need.

## On supporting arm64 AND amd64

Only the release builds are multi-platform, supporting `linux/arm64`
and `linux/amd64`. This enables higher level constructs like `helm` and
docker-compose to point to these images and effectively be multi-platform
as well.

Pull requests and master builds
are one-image-per-platform so that they can be parallized and the
build matrix for those is more sparse as we don't need to build every
build preset on every platform, and generally can be more selective here.
For those builds, we suffix tags with `-arm` where it applies.

### Working with Apple silicon

Apple's current generation of computers uses ARM-based CPUs, and Docker
running on MACs seem to require `linux/arm64/v8` (at least one user's M2 was
configured in that way). Setting the environment
variable `DOCKER_DEFAULT_PLATFORM` to `linux/amd64` seems to function in
term of leveraging, and building upon the Superset builds provided here.

```
export DOCKER_DEFAULT_PLATFORM=linux/amd64
```

Presumably, `linux/arm64/v8` would be more optimized for this generation
of chips, but less compatible across the ARM ecosystem.
