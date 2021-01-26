# SCC (SysConfCollect)

All the related scripts and files to install and manage SCC on both server and client environments.

## Compiling

To compile (for debian) execute the following scripts:

```shell script
# scc
sudo bash scc-src/scc_gen_all 1.26.73-rsync
# scc-srv
sudo bash scc-srv-src/scc-srv_gen_all 1.20.33-rsync
```

## Github flow publishes builds committed to master to the following locations:

```shell script
scc:        https://raw.githubusercontent.com/Driaan/hosting/master/scc_1.30.00-1_all.deb
scc-srv:    https://raw.githubusercontent.com/Driaan/hosting/master/scc-srv_1.30.00-1_all.deb
```

## Configuration

#### scc-srv

##### /etc/rsyncd.conf

```shell script
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid

[scc]
path = /var/opt/scc-srv/data/transfer/cp/
read only = no
uid = scc
gid = scc
hosts allow = *
auth users = scc
secrets file = /etc/rsyncd.scc.secrets
post-xfer exec = /opt/scc-srv/bin/scc-update
```

## Commands

#### Updating scc

```shell script
/opt/scc-srv/bin/scc-update -f
```

#### Adding a new realm

```shell script
/opt/scc-srv/bin/scc-realm -a $REALM_NAME
```

#### Deleting a realm

```shell script
/opt/scc-srv/bin/scc-realm -d $REALM_NAME
```

#### Adding hosts to realm

```shell script
scc-realm --add --quick --list $HOST_1 $REALM_NAME # do not call scc-update, manually call /opt/scc-srv/bin/scc-update -f
scc-realm --add --list $HOST_1,$HOST_2 $REALM_NAME
```

##### Renaming a realm

```shell script
/opt/scc-srv/bin/scc-update -r $OLD_REALM_NAME $NEW_REALM_NAME
```

##### Archiving hosts

```shell script
/opt/scc-srv/bin/scc-realm --archive /<dir>/archive --list $HOST_1,$HOST_2 --delete All
```

## Security

#### Adding user:password to realm via .htpasswd

Generating password: https://hostingcanada.org/htpasswd-generator/

```shell script
# driaan:123qwe

sudo tee -a /var/opt/scc-srv/data/www/.htpasswd > /dev/null <<EOT
driaan:{SHA}Bf50YcYHwzIpdy1AJQVgEBan0Oo=
hendrik:{SHA}3HJK8Y+91OWRifX+dopfgxFScFA=
EOT
```

#### Adding folder protection via .htaccess

```shell script
sudo tee -a /var/opt/scc-srv/data/www/$REALM_NAME/.htaccess > /dev/null <<EOT
AuthName "Access Control"
AuthType Basic
AuthUserFile /var/opt/scc-srv/data/www/.htpasswd
AuthGroupFile /var/opt/scc-srv/data/www/.htgroups
Require group admin
Require group $REALM_GROUP
EOT
```

Note: allowing users without needing groups: "Require valid-user"

#### Adding group protection via .htgroups

Activating apache2 authz_groupfile

```shell script
a2enmod authz_groupfile
systemctl restart apache2
```

Groups File:

```shell script
# /var/opt/scc-srv/data/www/.htgroups
$GROUP_NAME: $GROUP_USER_1 $GROUP_USER_2 $GROUP_USER_3

example:
admin: driaan hendrik
paythem: user_1 user_2
```

## Misc

```bash



find the related scripts in /scripts

older notes:

man rsyncd.conf
refuse options = in-plcae

post-xfer exec

refuse partial

as a test use --partial on big file and break its progress. have a look at whats been copied.
need to double check that it is stored as hidden, unique partial files.

```
