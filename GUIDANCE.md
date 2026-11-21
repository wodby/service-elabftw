# eLabFTW on Wodby

What Wodby sets up for eLabFTW on this service. eLabFTW runs from the official `elabftw/elabimg` image, which serves the web interface and the API with nginx and PHP-FPM on port 8080 and runs its own scheduler for email notifications and webhooks. It runs as one replica, as a non-root user with a read-only root file system.

## Linked services

| Link | Variables |
| --- | --- |
| MySQL (`mysql`, required) | `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_CERT_PATH` |
| SMTP server (`sendmail`, optional) | `ELABFTW_SMTP_HOST`, `ELABFTW_SMTP_PORT`, `ELABFTW_MAIL_FROM` |

eLabFTW needs MySQL 8 or later. It does not support MariaDB, and the service refuses to start on a MariaDB or older MySQL server. `DB_CERT_PATH` is the managed database server's CA certificate when it has one, so eLabFTW connects over TLS; it is empty for the bundled MySQL service.

eLabFTW keeps its email settings in the database and edits them in **Sysadmin panel > Email**. With the SMTP link, each start replaces the installer's placeholder SMTP server and sender address (`noreply@` the service's domain) with the linked server. Settings already saved in the Sysadmin panel are kept.

## What the manifest sets

- `SECRET_KEY` is built at start from the generated `key_bytes` token, in eLabFTW's key format. It encrypts the SMTP and timestamping passwords and webhook secrets stored in the database. Changing the token makes them unreadable; enter them again afterwards.
- `SITE_URL` is the service's URL in the environment. Links in emails and API responses use it.
- TLS ends at Wodby's router, which sets `X-Forwarded-Proto`, so `DISABLE_HTTPS` is `true`. `SET_REAL_IP` takes client addresses from `X-Forwarded-For` sent from private network addresses.
- `AUTO_DB_UPDATE` is `true`: a new version updates the database schema before eLabFTW serves requests.
- `MAX_PHP_MEMORY` is `512M` per request and `PHP_MAX_CHILDREN` is `10`, instead of eLabFTW's defaults of 2 GB and 50 processes, which are far beyond the container's memory limit. Raise the limit or lower these values if large exports or PDF generation run out of memory. `PHP_TIMEZONE` is `UTC`.

Uploaded files and exports are stored on the `data` volume at `/var/lib/elabftw`. Sessions are kept in the container, so a restart signs everyone out.

## First sysadmin

On the first deployment, the database is empty and eLabFTW is installed with a sysadmin from the Admin email, Admin password, Admin first name and Admin last name settings. The password must have at least 24 characters, with an uppercase letter, a lowercase letter, a digit and a special character. A weaker password stops the deployment before anything is installed. Changing the settings later does not change an existing account; change the password in eLabFTW.

eLabFTW's own sign-up page stays open by default, and new accounts wait until an administrator validates them. Both are settings in the Sysadmin panel.

## Changing configuration

Add or change eLabFTW's environment variables on the service and deploy it. Instance settings, such as email, storage, sign-in methods and teams, are managed in the Sysadmin panel and stored in the database.

## Upgrades

eLabFTW updates its database schema when a newer version starts. Back up the MySQL database and the `data` volume first; moving back to an older version is not supported.

## Check the result

From inside the environment, `/healthcheck` on port 8080 answers when nginx runs, and `/healthcheck.php` answers when eLabFTW can query the database. The service is probed with both.
