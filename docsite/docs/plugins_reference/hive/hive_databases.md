# hive_databases

## Synopsis

Provide a list of HIVE databases required by the application. They will be created (or destroyed) by HADeploy.

## Attributes

Each item of the list has the following attributes:

Name | req? |	Description
--- | --- |  ---
name |yes|The name of the HIVE database. 
properties |no | A map of properties. Equivalent to WITH DBPROPERTIES Hive DDL clause
location   |no | Equivalent to LOCATION Hive DDL clause
owner      |no | The owner of the database. May be a user account or a group.
owner_type |no | USER or ROLE. Specify what represent the owner attribute
comment    |no | Equivalent to the COMMENT Hive DDL close
no_remove|no|Boolean: Prevent this database to be removed when HADeploy will be used in REMOVE mode.<br>Default: `no`
ranger_policy|no|Definition of Apache Ranger policy bound to this database.<br>Parameters are same as [hive_ranger_policies](../ranger/hive_ranger_policies) except than `databases`, `tables` and `columns` should not be defined. The policy will apply on all columns of all tables of this database.<br>The policy name can be explicitly defined. Otherwise, a name will be generated as "`_<database>_`".<br>See example below for more information
when|no|Boolean. Allow [conditional deployment](../../more/conditional_deployment) of this item.<br>Default `True` 

### Example:

```yaml
hive_databases:
- name: jdctest1
```
Will internally generate the command:

```sql
CREATE DATABASE jdctest1
```
And

```yaml
hive_databases:
- name: jdctest2
  location: /user/apphive/db/testtables1
  owner_type: USER
  owner: apphive
  comment: "For jdchive table testing"
```
Will internally generate the commands:

```sql
CREATE DATABASE jdctest2 COMMENT 'For jdchive table testing' LOCATION 'hdfs://clusterid/user/apphive/db/testtables1'
ALTER DATABASE jdctest2 SET OWNER USER apphive
```
The following example illustrate the association with a Ranger policy, granting full rights on all table of this databases for the 'admin' user:

```yaml
hive_databases:
- name: jdctest3
  owner_type: USER
  owner: apphive
  ranger_policy:
    permissions:
    - users:
      - admin
      accesses:
      - all
      delegate_admin: yes
```


### Database Ownership

Database owner can be explicitly set by the `owner` attribute defined above. If this attribute is not present, then the owner will be:

* If kerberos is enabled, the account defined by the `hive_relay.principal` in the [`hive_relay`](./hive_relay) definition.
 
* If kerberos is not enabled, the account defined by the `ssh_user` set for the [`host`](../inventory/hosts) used as `hive_relay`

Once created, one may change owner by setting the corresponding attribute. Launching HADeploy with another ansible_user/principal will have no effect


