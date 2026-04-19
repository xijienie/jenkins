# Legacy SSH Release Pattern

This pattern matches the release jobs you shared:

- checkout selected branch
- compile with a fixed JDK
- publish the built artifact through `sshPublisher`
- restart service remotely
- support serial or parallel release

## This approach is valid

It is a practical migration step when your environment still depends on:

- Jenkins-managed SSH host configs
- remote shell restart scripts
- environment-specific WAR or JAR publish directories
- operator-selected host lists

## What should be improved

- Do not build on `master` or the controller node long term
- Do not hardcode repo URL, credential ID, and remote path in every job
- Do not duplicate the serial and parallel publish logic per service
- Do not treat rollback as a normal rebuild from branch code

## Recommended rules

- Use `build-jdk8` or `build-jdk17` labels for packaging
- Use `sshPublisher` only in the deploy stage
- Archive artifacts before deploy
- For rollback, deploy a saved artifact rather than rebuilding current branch code
- Keep `sourceFiles`, `remoteDirectory`, and `execCommand` in service metadata
- Normalize `start_service.sh` arguments to `paths war isolate mysql disconf`
- Reject deploy when isolate flag is not `0` or `1`

## Host list handling

`params.hostlist.split(',')` already works for one host and multiple hosts.
That means you do not need separate "normal" and "big version" stages if the publish behavior is the same.

## Best migration target

Short term:

- keep `sshPublisher`
- keep the remote restart scripts
- move job definition into Pipeline code

Mid term:

- replace hardcoded job values with catalog metadata
- introduce a shared publish function
- split build and deploy onto clearer node labels

## Account implementation details

- `notify` stage calls `send_build_start_v2.sh` through `notify_build.sh`
- `deploy` stage calls `start_service_compat.sh` in `execCommand`
- `start_service_compat.sh` supports both argument orders:
- new order: `isolate mysql disconf`
- old order: `mysql disconf isolate`
