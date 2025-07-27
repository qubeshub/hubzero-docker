# Steps to set up a QUBES development environment

(1) Install `ddev`

https://ddev.com/get-started/

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
mkdir -p public/app && cp -R app/config public/app/
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

This will open a web browser and, if you didn't change the DDEV project name, go to https://qubeshub.ddev.site. If you did change the DDEV project name, the site will be located at `https://<project name>.ddev.site` (e.g. https://hubzero.ddev.site if you ran alternate commands above for a Hubzero container with no QUBES template/configuration).

## Admin credentials

The administrative interface is located at https://qubeshub.ddev.site/administrator. The login credentials are:

**Username**: `admin` <br />
**Password**: `admin314`

Note: If the above password doesn't work, try the password `vagrant2016`.

## Create a non-admin user/account

It is best practice when logging in to the front-end website for testing and development to use a non-admin account. Here are the steps to create a new non-admin user/account.

(1) Go to https://qubeshub.ddev.site/register.

(2) Complete the form to register for an account (you can use a real email here -- there will be future instructions on how to set up mail capabilities for development and testing).

(3) Log into the [administrator interface](#admin-credentials). Since mail capabilities aren't working yet, you'll have to manually confirm your new user using the admin account.

(4) Go to Users -> Members in the administrator interface, find the new user, and hover over the caret next to "Unconfirmed" in the STATUS column. Choose "Confirm email". The status should change to "Not Approved".

(5) Hover over the caret next to "Not Approved" and choose "Approve". You should now be able to log in to the front-end as the new user.

## Database access

For a more detailed account of database management using DDEV, see https://ddev.readthedocs.io/en/stable/users/usage/database-management/.

### MacOS

If you are using [Sequel Ace](https://sequel-ace.com/), from the terminal within your project, you can load the `example` database with the following command:

```bash
ddev sequelace example
```

### WSL2/Linux

If you are using [DBeaver](https://dbeaver.io/), from the terminal within your project, you can load the `example` database with the following command:

```bash
ddev dbeaver example
```

### VSCode with DevDB extension

The [DevDB](https://marketplace.visualstudio.com/items?itemName=damms005.devdb) extension gives you access to the database directly from VSCode. In order for this to work, you need to create a `.devdbrc` file within your project root (either `qubeshub` or `qubeshub/public`, depending on which folder was added to your VSCode project):

```
[
        {
                "name": "ddev",
                "type": "mysql",
                "host": "127.0.0.1",
                "port": "<insert port here>",
                "username": "root",
                "password": "root",
                "database": "example",
                "options": {
                        "trustServerCertificate": true
                }
        }
]
```

**Specifying the port**: `ddev` uses dynamic ports for the database, which is why the port number is not specified above. To find which port is being used, you can type `ddev status` and grab the `Host` port information in the `URL/PORT` column of the `db` row. For example, the table cell might say:

```
InDocker -> Host:
 - db:3306 -> 127.0.0.1:32864
```

In this case, our database port is 32864, and the resulting line in the `.devdbrc` file should be:

```
                ...
                "port": "32864",
                ...
```

**Zero-config**: DevDB does have the ability to automatically figure out database information with DDEV projects (they call it "zero-config"), but it doesn't work in our case as it seems to (1) try to connect to the default `db` database, which we're not using, and (2) the username and password is incorrect.

**Static port**: We _could_ make the database port static in the `.ddev/config.yaml` using the [`host_db_port`](https://ddev.readthedocs.io/en/stable/users/configuration/config/#host_db_port) command to alleviate our need to change the `.devdbrc` database port everytime we restart the container, but this could become a problem on other machines or if additional concurrent containers are created.

# Additional information

## Steps to setup a supergroup dev environment

This section is for developers of supergroups on QUBES. We will assume that your supergroup already exists on QUBES and you have developer access to the resulting [GitLab](https://gitlab.hubzero.org/) repository. If not, please [submit a support ticket](https://qubeshub.org/support/ticket/new) requesting access.

(0) Once you have developer access to GitLab, [log in to GitLab](https://gitlab.hubzero.org/). Go to the remote repository for your supergroup, e.g. https://gitlab.hubzero.org/hub-qubeshub/sg_mysupergroup, replacing `mysupergroup` with the shortname for your group on QUBES (e.g. if you access your supergroup on QUBES via https://qubeshub.org/community/groups/mysupergroup, then your shortname would be `mysupergroup` and the above GitLab URL would work). If you have direct push access to this repository, then you can clone this one in Step 10 below. Otherwise, fork this repository, and clone the forked repository in Step 10 below.

(1) [Follow the steps above](#steps-to-set-up-a-qubes-development-environment) to set up a QUBES development environment.

(2) [Follow the steps above](#create-a-non-admin-useraccount) to create a new non-admin user account if you have not already done so.

(3) Log in to the front-end (https://qubeshub.ddev.site) with your non-admin user account.

(4) [Create a new group](https://qubeshub.ddev.site/community/groups/new), using the same Group ID as on QUBES (e.g. if you access your supergroup on QUBES via https://qubeshub.org/community/groups/mysupergroup, then your Group ID should be `mysupergroup`).

(5) Log in to the [administrative interface](https://qubeshub.ddev.site/administrator) using [the admin login credentials](#admin-credentials).

(6) Go to Users -> Groups, find the row associated with your new group, and write down the **4-digit number** in the GROUP ID column (yeah, I know, it's confusing that the Group ID field when creating a group is a word/string, while GROUP ID on the back-end is a numeric ID. _Technically_ throughout the codebase, and in the database, `id` refers to the numeric ID, and `cn` refers to the shortname, e.g. `mysupergroup`).

(7) Edit the new group on the back-end by either clicking the name or checking the box and selecting "Edit". In the DETAILS pane at the very top, change the Type from `Hub` to `Super` and click the "Save & Close" star icon in the upper right (Note: You can ignore the plethora of text in the yellow warning box).

(8) From the main DDEV project directory (e.g. `qubeshub`), go to the `public/app/site/groups` subdirectory (this is where all group file uploads and supergroup templates are located).

(9) Using the **numeric** group id you wrote down from before, delete the subdirectory `public/app/site/groups/<group id>`. For example, if your numeric group id is 1234, then the subdirectory to delete is `public/app/site/groups/1234` (BE CAREFUL! You might instead opt to just move the directory to something like `_1234`). We are going to replace it with a local version of the GitLab repository in the next step.

(10) Either from a terminal or using a Git GUI, clone the GitLab repository in Step 0 into the subdirectory you just deleted, `public/app/site/groups/<group id>`. For example, if you are using a terminal and are in the subdirectory `public/app/site/groups`, the command would be something like: `git clone https://gitlab.hubzero.org/hub-qubeshub/sg_simiode.git ./<group id>`. If it is a fork, then replace `hub-qubeshub` with your GitLab username.

That's it! When you access your supergroup from the development machine (e.g. https://qubeshub.ddev.site/community/groups/mysupergroup), it should show your template from the GitLab repository. You can now make changes and commit them to your local repository at `qubeshub\public\app\site\groups\<group id>`, pushing up to the remote repository when you are ready.

### Pulling code updates from GitLab into QUBES

Once you have pushed any changes/commits to the remote repository, you'll need to: 

 - Merge those changes if you forked the repository in Step 0 (this is because all supergroups on the server point to the `hub-qubeshub` owner on GitLab)
 - If you have QUBES administrator access:
   - Log in to the QUBES administrator interface
   - Go to Users -> Groups
   - Click the checkbox next to your supergroup and then click the "Update Groups Code" icon in the upper right
   - Click the merge button
 - If you do not have QUBES administrator access:
   - [Submit a support ticket](https://qubeshub.org/support/ticket/new) requesting someone pull in your code updates to the supergroup on QUBES

## Performance and file syncing

For a more detailed account of DDEV performance, see https://ddev.readthedocs.io/en/stable/users/install/performance/ (much of the content below in this section comes directly from this linked page).

DDEV by default uses [Mutagen](https://mutagen.io/) to do asynchronous caching of files with the mounted Docker container. DDEV will by default use Mutagen for MacOS and Windows/WSL2, but not Linux. Some things to keep in mind with Mutagen:

- **Avoid file changes when DDEV is stopped**

  If you change files while DDEV is stopped, run `ddev mutagen reset` _before_ restarting the project so Mutagen only starts with awareness of the host’s file contents.

- **Syncing after `git checkout`**

  In general, it’s best practice on most projects to do significant Git operations on the host, but they can be disruptive to the sync. It’s easy to add a Git post-checkout hook to do a `ddev mutagen sync` operation though. Add a `.git/hooks/post-checkout` file to your project and make it executable with `chmod +x .git/hooks/post-checkout`:

  ```bash
  #!/usr/bin/env bash
  ddev mutagen sync || true
  ```

## Understanding DDEV configuration

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

## PHP Debugging

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

## Solr Search

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

## What to do on restarting ddev

All code, static files, and databases should persist across (1) `ddev restart`, (2) `ddev stop` then `ddev start`, or even (3) `ddev delete` then `ddev start` (assuming you don't include `--omit-snapshot` with `ddev delete`, which will destroy the databases as well). Note, however, that Solr data will be deleted, so you will have to rebuild indexes. In the admin interface, go to `Components -> Search`, choose `Searchable Components`, and rebuild the desired indexes (in particular, make sure to rebuild `Publications` for resource search & browse).

## Migrating from vagrant

TBD
