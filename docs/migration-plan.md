# Migration Plan

## Why rebuild Jenkins now

- The current Jenkins version is too old for safe long-term maintenance
- Plugin versions and job configuration are inconsistent
- Environment groups `devc`, `devd`, `devg`, and `devk` are managed differently
- The maintenance cost is inside Jenkins UI instead of Git

## Target State

- One new Jenkins controller on a dedicated host for non-production engineering delivery
- All controller configuration stored as code
- One shared pipeline model for all service repositories
- One service catalog to describe repo, branch, JDK, node labels, and deploy target
- Environment differences handled by metadata, not copied jobs

## Migration Phases

### Phase 1: Build the new control plane

- Prepare a clean VM or physical host for the new Jenkins controller
- Create a standard directory layout under `/data/jenkins`
- Install Jenkins LTS, JDKs, Maven, Git, and the plugin CLI on the host
- Enable Configuration as Code and required pipeline plugins
- Create managed credentials, JDK tools, and Maven tools
- Import the shared library and seed pipeline
- Copy the compatibility wrapper scripts to the Jenkins host

### Phase 2: Normalize service metadata

- Group repositories by `devc`, `devd`, `devg`, `devk`
- Record repo URL, default branch, JDK, packaging type, build node label, and deploy node label
- Mark exceptional projects such as old JDK8 or JDK11 workloads
- Record which old shell scripts each job still depends on

### Phase 3: Migrate by batch

- Start from 5 to 10 low-risk services
- Put the standard `Jenkinsfile` into those repositories
- Compare build output from old and new Jenkins
- Compare deployment behavior between old shell scripts and new wrapper scripts
- Switch webhook and team entrypoint after validation

### Phase 4: Remove UI-only jobs

- Freeze changes in the old Jenkins
- Move remaining freestyle or copied pipeline jobs into code
- Move shell scripts that are still useful from scattered hosts into `/data/jenkins/scripts`
- Retire legacy nodes and unused plugins

## Recommended Job Strategy

- Use multibranch pipeline where each repo has active branch work
- Use one standard `Jenkinsfile` per service repo
- Keep deployment switches parameterized
- If release orchestration is needed, build a second layer of environment promotion jobs
- For old XML Maven jobs, migrate first to Pipeline syntax, then optimize deployment scripts
- Separate build nodes from deploy nodes where possible

## Key Decisions To Confirm

- Whether the controller and agents are split across multiple hosts or kept on one large host at first
- Whether deploys target service hosts directly or still rely on existing shell orchestration
- Which JDK versions must be supported in parallel
- Whether old projects still need freestyle jobs during transition

## Minimum Standards

- All jobs from code
- All credentials managed centrally
- All plugins pinned and reviewed
- Shared build, test, archive, and deploy stages
- No environment-specific copied pipelines
- No hardcoded absolute workspace paths such as `/home/fr/jenkinsDicdevg/workspace/...`
- Old shell scripts must be called through managed wrapper scripts, not directly from UI job XML
