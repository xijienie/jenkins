# Environment Plan

This plan is for a host-installed Jenkins rebuild, not Docker.

## Core idea

Split the environment into three layers:

- Controller: stores Jenkins home, plugins, credentials, and JCasC
- Build nodes: compile code with controlled JDK and Maven versions
- Deploy nodes: run the legacy publish and restart scripts for each environment

## Recommended host layout

- `/data/jenkins/home`: `JENKINS_HOME`
- `/data/jenkins/config`: JCasC, plugin list, and seed job config
- `/data/jenkins/scripts`: managed wrapper scripts kept in Git and copied to the host
- `/data/jenkins/publish/jar`: publish area for jar artifacts
- `/data/jenkins/publish/war`: publish area for war artifacts
- `/data/jenkins/tools/jdk8`: JDK 8
- `/data/jenkins/tools/jdk17`: JDK 17
- `/data/jenkins/tools/maven-3.9`: Maven
- `/data/jenkins/logs`: controller and wrapper script logs

## Label plan

Replace old labels such as `home_devc` with stable roles:

- `build-jdk8`: jobs that must compile with JDK 8
- `build-jdk17`: jobs that compile with JDK 17
- `deploy-devc`: deploy jobs for the `devc` environment
- `deploy-devd`: deploy jobs for the `devd` environment
- `deploy-devg`: deploy jobs for the `devg` environment
- `deploy-devk`: deploy jobs for the `devk` environment

## Tool plan

- Jenkins controller runtime: JDK 17
- Build toolchains registered in Jenkins global tools:
- `jdk8`
- `jdk17`
- `maven-3.9`

## Script governance

The old scripts under `/home/fr/tools/shell` should not be called directly from Jenkins job UI.
Instead:

- Jenkins pipeline calls `/data/jenkins/scripts/notify_build.sh`
- Jenkins pipeline `sshPublisher` calls `/data/jenkins/scripts/start_service_compat.sh`
- `start_service_compat.sh` normalizes args and invokes `/data/jenkins/scripts/start_service_v2.sh`
- `start_service_v2.sh` keeps the old start flow but removes `checkService.sh`

This gives you one place to log, validate, and gradually replace the legacy behavior.

For `sshPublisher` execution, keep `/data/jenkins/scripts` synchronized on each deploy target host.

## Migration rule

- Build configuration belongs in `services.yaml`
- Pipeline logic belongs in the shared library
- Wrapper logic belongs in `/data/jenkins/scripts`
- Nothing important should live only inside Jenkins UI XML
