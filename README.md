# Jenkins Rebuild Blueprint

This repository now targets a host-installed Jenkins setup for `devc`, `devd`, `devg`, and `devk`.
The focus is not Docker. The focus is to clean up the environment, standardize node usage, and preserve the old shell deployment capability while moving job definitions into code.

## Goals

- Rebuild Jenkins on a dedicated VM or physical host
- Standardize build nodes, deploy nodes, tools, and publish directories
- Move job behavior from Jenkins UI XML into Git-managed pipeline code
- Keep compatibility with old scripts such as `send_build_start.sh` and `start_service.sh`
- Migrate `devc`, `devd`, `devg`, and `devk` by batch instead of all at once

## Repository Layout

- `jenkins/casc/jenkins.yaml`: Jenkins Configuration as Code for a host-installed controller
- `jenkins/casc/plugins.txt`: pinned plugin list
- `jenkins/catalog/services.yaml`: service inventory and legacy compatibility metadata
- `jenkins/jobs/platform.Jenkinsfile`: seed entry for the shared pipeline
- `jenkins/shared-library/vars/servicePipeline.groovy`: unified pipeline logic
- `jenkins/host/bootstrap-host.sh`: create the standard host layout
- `jenkins/host/jenkins.env.example`: controller runtime variables
- `jenkins/host/systemd/jenkins.service`: sample `systemd` unit
- `jenkins/host/scripts/notify_build.sh`: wrapper for old build notification script
- `jenkins/host/scripts/deploy_with_legacy.sh`: wrapper for old publish/start script
- `jenkins/host/scripts/send_build_start_v2.sh`: hardened build notification script with curl failure handling
- `jenkins/host/scripts/start_service_compat.sh`: argument normalization wrapper for old/new start script order
- `jenkins/host/scripts/start_service_v2.sh`: account start script without `checkService.sh` dependency
- `docs/environment-plan.md`: recommended environment and node planning
- `docs/legacy-ssh-release-pattern.md`: recommended structure for `sshPublisher` style release jobs
- `docs/migration-plan.md`: migration sequence
- `docs/xml-to-pipeline-mapping.md`: mapping from old XML jobs to the new model
- `templates/Jenkinsfile`: example per-repo pipeline entry
- `templates/legacy-ssh-release.Jenkinsfile`: example release pipeline close to your existing jobs

## Suggested Model

1. Build one new Jenkins controller on a clean host.
2. Create standard directories under `/data/jenkins`.
3. Standardize labels such as `build-jdk8`, `build-jdk17`, `deploy-devc`, `deploy-devd`, `deploy-devg`, `deploy-devk`.
4. Keep old deployment scripts, but call them through wrapper scripts owned by Git.
5. Migrate a few representative jobs first, then expand in batches.

## Assumptions

- Most services are Maven projects
- Part of the fleet still requires JDK 8, while newer services can use JDK 17
- A large part of the old release logic still lives in shell scripts on Jenkins nodes
- The old Jenkins should remain available during migration

Before you wire production credentials into Jenkins, replace placeholder repo URLs, credential IDs, tool names, and host paths.

## Account baseline implemented

- `servicePipeline` now supports account release parameters:
- `branch`, `paths`, `disconf`, `mysql`, `redis`, `deploy_env`, `release_mode`, `hostlist`
- deploy uses one serial/parallel implementation through `sshPublisher`
- argument normalization is handled by `start_service_compat.sh`
- account service metadata is centralized in `jenkins/catalog/services.yaml`
