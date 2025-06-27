# Steps to setup a dev environment

(1) Install `ddev`

[https://ddev.com/get-started/](https://ddev.com/get-started/)

(2) Clone this repository

```bash
git clone https://github.com/qubeshub/hubzero-docker.git ./qubeshub
cd qubeshub
```

<details>
<summary>
Alternate commands for a Hubzero container with no QUBES template/configuration
</summary>

```bash
git clone https://github.com/qubeshub/hubzero-docker.git ./hubzero
cd hubzero
```
</details>
<p></p>
(3) Install CMS
<p></p>

```bash
git clone https://github.com/qubeshub/hubzero-cms.git ./public
cp -r app/config public/app/config
ddev start
ddev import-db --file=data/databasedump_qubeshub.sql.gz --database=example
```

<details>
<summary>
Alternate commands for a Hubzero container with no QUBES template/configuration
</summary>

```bash
git clone https://github.com/hubzero/hubzero-cms.git ./public
cp -r app/config public/app/config
ddev config --project-name=hubzero
ddev start
ddev import-db --file=data/databasedump_hubzero.sql.gz --database=example
```
</details>
<p></p>
(4) Run setup scripts
<p></p>

```bash
ddev ssh
cd public/core
php bin/composer install
php bin/muse migration -f
exit
```

Note: The `migration` command will likely throw the following error, which you can ignore:
```
PHP Fatal error:  Uncaught Error: Call to undefined function Hubzero\Base\fastcgi_finish_request()
```

(5) Launch the site!

```bash
ddev launch
```

This will open a web browser and, if you didn't change the DDEV project name, go to `https://qubeshub.ddev.site`. If you did change the DDEV project name, the site will be located at `https://<project name>.ddev.site` (e.g. `https://hubzero.ddev.site` if you ran alternate commands above for a Hubzero container with no QUBES template/configuration).

# Admin credentials

The administrative interface is located at `https://qubeshub.ddev.site/administrator`. The login credentials are:

**Username**: `admin` <br />
**Password**: `vagrant2016`

# What to do on restarting ddev

All code, static files, and databases should persist across (1) `ddev restart`, (2) `ddev stop` then `ddev start`, or even (3) `ddev delete` then `ddev start` (assuming you don't include `--omit-snapshot` with `ddev delete`, which will destroy the databases as well). Note, however, that Solr data will be deleted, so you will have to rebuild indexes. In the admin interface, go to `Components -> Search`, choose `Searchable Components`, and rebuild the desired indexes (in particular, make sure to rebuild `Publications` for resource search & browse).

# Details

## DDEV Configuration

The following lists the additional configuration settings with their explanations.

- `.ddev/config.yaml`
  - `docroot: public`
  - `webserver_type: nginx-fpm`
  - `upload_dirs`: `app/site` and `../data/example` (So Mutagen doesn't sync these files - they will instead be mounted in Docker images). Note that the paths are relative to `docroot`.
  - `hooks`: Launch Solr
  - `web_extra_exposed_ports`: Access to Solr Admin at http://{project name}.ddev.site:8445
- `nginx_full/nginx-site.conf`: Redirect /administrator and /api to /index.php. Also turns off fastcgi buffering to avoid excessive PHP warnings.
- `mysql/sql-mode.cnf`: Make sure mysql strict mode isn't set so migrations can run (presumably can be removed after migration to reinstate strict mode)
- `web-build/Dockerfile`: Install hubzero-solr package
- `docker-compose.mounts.yaml`: Mount `data/example` directory to `/srv/example` inside the container. This is used for project files and will ensure persistence across ddev status updates (such as `ddev restart`, `ddev stop`, and even `ddev destroy`).

## Troubleshooting

The command `ddev logs` is very helpful and will show you PHP and database logs. If you want realtime log updates, include the `-f` flag. The following command shows realtime log updates and color codes warning (`[warn]`) and error (`[error]`) messages, filtering out all other message types, such as information-only (`[info]`):

```bash
ddev logs -f | stdbuf -oL grep -iE '\[warn\]|\[error\]' | stdbuf -oL perl -pe '
    s/\[warn\]/\e[33m[warn]\e[0m/i;
    s/\[error\]/\e[31m[error]\e[0m/i;
  '
```

Or, use the following if you want a bash function (e.g., include in your `.bashrc` file):

```bash
ddev_logs() {
  ddev logs -f | stdbuf -oL grep -iE '\[warn\]|\[error\]' | stdbuf -oL perl -pe '
    s/\[warn\]/\e[33m[warn]\e[0m/i;
    s/\[error\]/\e[31m[error]\e[0m/i;
  '
}
```

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
