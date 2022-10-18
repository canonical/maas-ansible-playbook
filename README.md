# MAAS-ansible-playbook
An Ansible playbook for installing and configuring MAAS

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

Example Host File:

```
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

## Run

This playbook requires a user to set the following set of variables:

- **maas_version**: The intended version of MAAS to install. Example: '3.2'

- **maas_postgres_password**: The password for the MAAS postgres user. Example: 'my_password'

- **maas_postgres_replication_password**: The password for the replication postgres user. WARNING this variable will be set by the playbook in stable releases.

- **maas_installation_type**: The intended type of MAAS installation. Possible values are: 'deb' or 'snap'

- **maas_url**: The MAAS URL MAAS will be accessed and rack controllers should use for registering with region controllers. Example: 'http://proxy01.example.com:5240/MAAS'

### Deploy the MAAS stack

```
ansible-playbook -i ./hosts\
    --extra-vars="maas_version=3.2 maas_postgres_password=example maas_postgres_replication_password=example maas_installation_type=deb maas_url=http://example.com:5240/MAAS\
    ./site.yaml"
```

### Teardown the MAAS stack

```
ansible-playbook -i ./hosts ./teardown.yaml
```


