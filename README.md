# fuseki-docker

Builds the official Apache Jena Fuseki Docker image and publishes it to
[`netwerkdigitaalerfgoed/fuseki`](https://hub.docker.com/r/netwerkdigitaalerfgoed/fuseki) on Docker Hub.

The image is built from Apache's own [`jena-fuseki-docker`](https://repo1.maven.org/maven2/org/apache/jena/jena-fuseki-docker/) recipe (Dockerfile + helper scripts) downloaded from Maven Central at build time. We do not maintain a Dockerfile of our own — the only thing we pin here is the **Jena version** in [`pom.xml`](pom.xml).

## How it works

1. [`pom.xml`](pom.xml) declares a single dependency on `org.apache.jena:jena-fuseki-docker` at the desired version.
2. [Dependabot](.github/dependabot.yml) watches that dependency and opens a pull request whenever Apache releases a new version of Jena.
3. The auto-merge workflow merges Dependabot PRs once CI passes.
4. The merge triggers [`.github/workflows/build.yml`](.github/workflows/build.yml), which:
   - downloads Apache's `jena-fuseki-docker-<version>.zip` from Maven Central,
   - runs `docker build` against the Dockerfile inside that archive (passing `JENA_VERSION=<version>`),
   - pushes a multi-arch (linux/amd64 + linux/arm64) image to Docker Hub with tags `<version>`, `<major>`, and `latest`,
   - creates a `v<version>` GitHub release with auto-generated notes.

The downstream `infrastructure` repo (Flux) tracks the published image tag and rolls Fuseki forward automatically.

## Image tags

- `netwerkdigitaalerfgoed/fuseki:6.1.0` — exact Jena version
- `netwerkdigitaalerfgoed/fuseki:6` — major channel
- `netwerkdigitaalerfgoed/fuseki:latest` — most recent build

## Manually triggering a build

GitHub → Actions → "Push image to Docker Hub" → Run workflow.

## Bumping the Jena version manually

Edit the `<version>` inside `pom.xml` and push to `master`. The workflow will build, publish, and release.

## Scope

This image contains **only the Fuseki server** — it does not bundle the Apache Jena command-line tools (e.g. `tdb2.xloader`, `tdb2.tdbloader`). Workloads that need those should download `apache-jena-<version>.tar.gz` from Maven Central at runtime.
