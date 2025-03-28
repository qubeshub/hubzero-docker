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
git clone https://github.com/qubeshub/hubzero-cms.git ./qubeshub
cd qubeshub
rm -rf app/config
cp -r ../config app/config
ddev config --php-version=5.6 --database=mysql:5.5
ddev start
ddev import-db --file=../databasedump.sql --database=example
```

(4) Run setup scripts

```bash
ddev ssh
cd core
php bin/composer install
cd ..
php muse migration --file=Migration20180126160144ComSearch.php -f
php muse migration -f
```
