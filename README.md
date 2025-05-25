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
git clone https://github.com/hubzero/hubzero-cms.git ./public
ddev config --php-version=8.2 --database=mariadb:10.11
ddev start
ddev import-db --file=databasedump.sql --database=example
ddev import-files --source=app
```

(4) Run setup scripts

```bash
ddev ssh
cd public/core
php bin/composer install
php bin/muse migration -f
```
