# MAAS-ansible-playbook
An Ansible playbook for installing and configuring MAAS, further documentation is found [here](https://maas.io/docs/ansible-playbooks-reference).

## Supported versions
This playbook has been tested with Ansible version 5.10.0 and above. We recommend using the latest available stable version of Ansible (currently 7.x). The `netaddr` Python library needs to be installed on the machine on which Ansible is used; note that this is not required on remote hosts.

## Install

```
git clone git@github.com:maas/maas-ansible-playbook
```

## Setup
This playbook has several main roles, each has a corresponding group to assign hosts to. They are as follows:

- **maas_postgres_primary**: This role and group is for a host to be configured as an initial Postgres primary instance for the MAAS stack. This role is required.

- **maas_postgres_secondary**: This role and group is for hosts to be configured as Postgres secondaries in hot standby with synchronous replication. This role is optional.

- **maas_region_controller**: This role and group is for hosts to be configured as MAAS Region Controllers. This role is required at least once.

- **maas_rack_controller**: This role and group is for hosts to be configured as MAAS Rack Controllers. This role is required at least once.

- **maas_proxy**: This role and group is for hosts to be configured as an HAProxy instance in front of region controllers for HA stacks. This role is optional.

### Host file

Two formats can be used to define the inventory (`ini` and `yaml`). Below, the same inventory can be found twice: defined in both formats. Only one is to be chosen when running the playbook.  

More information can be found on the [Ansible inventory documentation page](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html).

#### ini

```ini
[maas_postgres_primary]
db01.example.com

[maas_postgres_secondary]
db02.example.com
db03.example.com

# DO NOT MODIFY THIS GROUP
[maas_postgres:children]
maas_postgres_primary
maas_postgres_secondary

[maas_region_controller]
region01.example.com
region02.example.com
region03.example.com

[maas_rack_controller]
region01.example.com
rack01.example.com
rack02.example.com

[maas_proxy]
proxy01.example.com
```

#### yaml

```yaml
---
all:
  children:
    maas_postgres:
      children:
        maas_postgres_primary:
          hosts:
            db01.example.com:
        maas_postgres_secondary:
          hosts:
            db02.example.com:
            db03.example.com:
    maas_proxy:
      hosts:
        proxy01.example.com:
    maas_region_controller:
      hosts:
        region01.example.com:
        region02.example.com:
        region03.example.com:
    maas_rack_controller:
      hosts:
        region01.example.com:
        rack01.example.com:
        rack02.example.com:
```

## Run

This playbook requires a user to set the following set of variables:

- **maas_version**: The intended version of MAAS to install.\
  Example: `'3.2'`

- **maas_postgres_password**: The password for the MAAS postgres user.\
  Example: `'my_password'`

- **maas_installation_type**: The intended type of MAAS installation.\
  Possible values are: `'deb'` or `'snap'`


- **maas_url**: The MAAS URL where MAAS will be accessed and which rack controllers should use for registering with region controllers.\
  Example: `'http://proxy01.example.com:5240/MAAS'`


There are additional optional variables that can be passed to the playbooks:

- [Admin credentials] [If setting up an admin user]

  - **admin_username**: The username of the admin user\
    Default: `admin`

  - **admin_password**: The password of the admin user\
    Default: `admin`

  - **admin_email**: The email address of the admin user\
    Default: `admin@email.com`

  - **admin_id**: The launchpad id of the admin user\
    Default: `admin`

- **maas_proxy_postgres_proxy_enabled**: Use postgres proxy uri\
  Default: `false`

- **install_metrics**: Whether MAAS should install metrics\
  Default: `false`

- **enable_tls**: Whether MAAS should enable TLS\
  Default: `false` [Only valid for MAAS >= 3.2]

- [MAAS Vault] [Only valid for MAAS >= 3.3]

  - **vault_integration**: Whether MAAS Should use Vault for secret storage\
    Default: `false`
  - **vault_url**: The URL of the MAAS Vault\
    Default: undefined
  - **vault_approle_id**: The approle id of the MAAS Vault\
    Default: undefined
  - **vault_wrapped_token**: The wrapped token for the MAAS Vault\
    Default: undefined
  - **vault_secrets_path**: The secrets path of the MAAS Vault\
    Default: undefined
  - **vault_secret_mount**: The secret mount of the MAAS Vault\
    Default: undefined

- **http_proxy**: The HTTP Proxy to use\
  Default: proxy not used

- **https_proxy**: The HTTPS Proxy to use\
  Default: proxy not used

### Deploy the MAAS stack

```
ansible-playbook -i ./hosts\
    --extra-vars="maas_version=3.2 maas_postgres_password=example maas_installation_type=deb maas_url=http://example.com:5240/MAAS"\
    ./site.yaml
```

### Teardown the MAAS stack

```
ansible-playbook -i ./hosts ./teardown.yaml
```

### Backup the MAAS stack

Backup MAAS requires the `maas_backup_download_path` variable to be set, it will designate a path local to where the playbook is being run to download backup archives. This path must exist prior to running the playbook. `maas_installation_type` is also required.

```
ansible-playbook -i ./hosts --extra-vars="maas_backup_download_path=/tmp/maas_backups/ maas_installation_type=deb" ./backup.yaml
```

### Restore from Backup

Since backup is per-host, it is recommended to set the `maas_backup_file` variable as a host variable, or filter execution to a specific host, but `maas_backup_file` should be set to a gzipped tar file from the backup action and `maas_installation_type` should be set as well.

The below example assumes `maas_backup_file` is set in `./hosts`

```
ansible-playbook -i ./hosts --extra-vars="maas_installation_type=deb" ./restore.yaml 
```

### Create a new admin user

For an already existing MAAS installation, a new admin user can be created. The below example assumes the host for which the account is to be created is set under `[maas_region_controller]` in `./hosts`. `user_ssh` allows for upload of a Launchpad (`lp:id`) or GitHub (`gh:id`) public key, or can be left blank (`user_ssh=`). 

```
ansible-playbook -i ./hosts --extra-vars="user_name=newuser user_pwd=newpwd user_email=user@email.com user_ssh=lp:id" ./createadmin.yaml
```
