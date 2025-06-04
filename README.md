# Steps to setup a dev environment

(1) Install `ddev`

[https://ddev.com/get-started/](https://ddev.com/get-started/)

(2) Clone this repository

```bash
git clone https://github.com/qubeshub/hubzero-docker.git ./hubzero-docker
cd hubzero-docker
```

(3) Install CMS

```bash
git clone https://github.com/qubeshub/hubzero-cms.git ./public
cp -r app/config public/app/config
ddev config --php-version=8.2 --database=mariadb:10.11
ddev start
ddev import-db --file=databasedump.sql --database=example
```

(4) Run setup scripts

```bash
ddev ssh
cd public/core
php bin/composer install
php bin/muse migration -f
exit
```

(5) Launch the site!

```bash
ddev launch
```

# DDEV Configuration

The following lists the additional configuration settings with their explanations.

- `.ddev/config.yaml`
  - `docroot: public`
  - `webserver_type: nginx-fpm`
  - `upload_dirs: app/site` (So Mutagen doesn't sync these files - they will instead be mounted in a Docker image)
  - `hooks`: Launch Solr
  - `web_extra_exposed_ports`: Access to Solr Admin at http://<project name>.ddev.site:8445
- `nginx_full/nginx-site.conf`: Redirect /administrator and /api to /index.php
- `mysql/sql-mode.cnf`: Make sure mysql strict mode isn't set so migrations can run (presumably can be removed after migration to reinstate strict mode)
- `web-build/Dockerfile`: Install hubzero-solr package

# Details

## Solr

### Configuration

Solr configuration in admin Search settings:
- Solr Host: `localhost`
- Solr Port: `8445`
- Solr Core: `hubzero-solr-core`
- Solr Context: `solr`
- Solr Path: `/`
- Solr Log Path: `/srv/hubzero-solr/logs/solr.log`

### Troubleshooting

When attempting to access Solr Admin, you may receive an Error 431: Request Header Fields Too Large. Here are the steps to fix this:

- `ddev ssh`
- `sudo vi /etc/default/hubzero-solr.in.sh`
- Add `SOLR_OPTS="$SOLR_OPTS -Dsolr.jetty.request.header.size=65535"` and save the file.
- `sudo /etc/init.d/hubzero-solr restart`

## Migrating from vagrant

TBD