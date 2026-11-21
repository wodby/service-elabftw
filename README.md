# eLabFTW service for Kubernetes on Wodby

Run eLabFTW as a reusable Kubernetes application service with Wodby.

This repository defines the Wodby service manifests and operational
configuration for eLabFTW.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [eLabFTW application stack](https://github.com/wodby/stack-elabftw)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `elabftw` |
| Type | Application service |
| Versions | `6.0` |
| Workloads | `main` (StatefulSet, primary) |
| Containers | `elabftw` using `elabftw/elabimg` |
| Endpoints | `elabftw`: HTTP 8080 (main) |
| Service links | MySQL (`mysql`), required; SMTP server (`sendmail`), optional |
| Application build | Not buildable from application source |
| Helm | chart `oci://registry-1.docker.io/wodby/stateful`; version `0.4.0` |
| Configuration and operations | 4 settings, 1 volume |

## Use this service

Use this service through [eLabFTW application stack](https://github.com/wodby/stack-elabftw), or reference `elabftw` from a
custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Database

eLabFTW needs MySQL 8 or later. It does not support MariaDB, and the service
refuses to start on a MariaDB or older MySQL server.

## First sysadmin

The first deployment installs eLabFTW with a sysadmin from the service's
required Admin email and Admin password settings, so the public sign-up page
never makes a stranger the sysadmin. The password must have at least 24
characters, with an uppercase letter, a lowercase letter, a digit and a special
character. Changing the settings later does not change an existing account.

## Upgrades

eLabFTW updates its database schema when a newer version starts. Back up the
MySQL database and the uploaded files first; moving back to an older version is
not supported.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
