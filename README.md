# fuseki-docker

Builds two Docker images from a single pinned Jena version and publishes them to Docker Hub:

- [`netwerkdigitaalerfgoed/fuseki`](https://hub.docker.com/r/netwerkdigitaalerfgoed/fuseki) — the Apache Jena **Fuseki server**.
- [`netwerkdigitaalerfgoed/jena-tools`](https://hub.docker.com/r/netwerkdigitaalerfgoed/jena-tools) — the Apache Jena **command-line tools** (`tdb2.xloader`, `tdb2.tdbloader`, `jena.textindexer`, …) for loader / batch jobs.

The server image is built from Apache's own [`jena-fuseki-docker`](https://repo1.maven.org/maven2/org/apache/jena/jena-fuseki-docker/) recipe downloaded from Maven Central at build time; the tools image is built from our own [`jena-tools/Dockerfile`](jena-tools/Dockerfile). Both take the **Jena version** from a single source — [`pom.xml`](pom.xml) — so they are always released in lockstep and the TDB2 / Lucene formats the tools write match exactly what the server reads.

## How it works

1. [`pom.xml`](pom.xml) declares a single dependency on `org.apache.jena:jena-fuseki-docker` at the desired version.
2. [Dependabot](.github/dependabot.yml) watches that dependency and opens a pull request whenever Apache releases a new version of Jena.
3. The auto-merge workflow merges Dependabot PRs once CI passes.
4. The merge triggers [`.github/workflows/build.yml`](.github/workflows/build.yml), which builds **both** images at that one version:
   - **server** — downloads Apache's `jena-fuseki-docker-<version>.zip` from Maven Central and `docker build`s the Dockerfile inside it (`JENA_VERSION=<version>`),
   - **tools** — `docker build`s [`jena-tools/Dockerfile`](jena-tools/Dockerfile) (same `JENA_VERSION`), which lays the `apache-jena-<version>` CLI tools onto a GNU-coreutils base,
   - pushes both as multi-arch (linux/amd64 + linux/arm64) images tagged `<version>`, `<major>`, and `latest`,
   - creates a `v<version>` GitHub release with auto-generated notes.

The downstream `infrastructure` repo (Flux) tracks the published image tags and rolls both forward automatically.

## Image tags

Both images share the same tag scheme (`<version>`, `<major>`, `latest`) and are always published together at the same Jena version:

- `netwerkdigitaalerfgoed/fuseki:6.1.0` / `:6` / `:latest` — the Fuseki server
- `netwerkdigitaalerfgoed/jena-tools:6.1.0` / `:6` / `:latest` — the Jena CLI tools

## Manually triggering a build

GitHub → Actions → "Push image to Docker Hub" → Run workflow.

## Bumping the Jena version manually

Edit the `<version>` inside `pom.xml` and push to `master`. The workflow will build, publish, and release.

## Scope

The **`fuseki`** image contains only the Fuseki server. Loader / batch workloads that need the Jena command-line tools (`tdb2.xloader`, `tdb2.tdbloader`, `jena.textindexer`, …) should use the **`jena-tools`** image instead.

### Why `jena-tools` pins a GNU-coreutils base

`tdb2.xloader` drives the load through `bash`, GNU `sort` (it uses `--parallel` / `--compress-program`) and `jq`. Two common bases **silently corrupt** the load and must be avoided:

- **Alpine** — busybox `ash`/`sort` (no `bash`, non-GNU `sort`).
- **Ubuntu 25.10+ / plain `eclipse-temurin:21-jre`** — now ships `uutils-coreutils`, whose `sort` breaks xloader (node-table `Bad hex char`, `sort RC = 141`).

So [`jena-tools/Dockerfile`](jena-tools/Dockerfile) pins `eclipse-temurin:21-jre-noble` (Ubuntu 24.04 LTS, GNU coreutils). This was verified by loading the full GeoNames dataset (~137.7M triples) end-to-end.
