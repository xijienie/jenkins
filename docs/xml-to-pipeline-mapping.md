# XML To Pipeline Mapping

This document maps the old Jenkins job XML pattern to the new Jenkins pipeline design.

## What the sample XML tells us

- Job type: `maven2-moduleset`
- Build style: parameterized branch build
- Runtime: fixed node like `home_devc`
- Toolchain: `jdk1.8`
- Build command: `clean package -Pdeve -DskipTests -U -e`
- Deploy style: shell script after build
- Artifact move: manual `mv` into a publish directory
- Environment isolation: selected by `paths`, `disconf`, `mysql`, `redis`

## Migration Mapping

| Old XML field | Meaning | New location |
| --- | --- | --- |
| `<assignedNode>` | Fixed build node | `buildNodeLabel` and `deployNodeLabel` in `services.yaml` |
| `<jdk>` | JDK version | `jdk` in `services.yaml` |
| `<url>` + `<credentialsId>` | SCM source | `repo` plus Jenkins managed credential |
| `<name>${branch}</name>` | Dynamic branch | `branch` pipeline parameter |
| `<goals>` | Maven build command | `buildGoals` in `services.yaml` |
| `<prebuilders>` | Start notification | `notify_build.sh` wrapper plus `legacy.notifyScript` |
| `<postbuilders>` | Deploy and restart | `sshPublisher` exec with `start_service_compat.sh` plus `legacy.startScript` |
| `paths/disconf/mysql/redis` | Runtime selectors | `legacy.parameters.*` |

## Important cleanup items

- Replace old Jenkins credential UUIDs with readable IDs
- Remove absolute workspace path references and use `$WORKSPACE`
- Stop using Jenkins Maven project jobs and switch to Pipeline jobs
- Move critical wrapper logic into Git, then slowly absorb old scripts from `/home/fr/tools/shell`
- Separate build and deploy where possible so builds stay reproducible

## Recommended next batch

1. Pick 3 representative services: one pure jar, one war, one special deployment job.
2. Export their XML configs and extract common fields.
3. Put those common fields into `services.yaml`.
4. Convert their shell deploy logic into reusable wrapper scripts and pipeline steps.
5. Validate on the new Jenkins before moving the long tail.
