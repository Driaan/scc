# SCC (SysConfCollect)

The source code for SCC (SysConfCollect) on both server and client environments.

## Releases

Releases are automatically built and published via GitHub Actions to this repository.

## Compiling

To manually compile (tested on debian) execute the following scripts:

```shell script
# scc
sudo bash scc/scc_gen_all 1.31.0-rsync-proxmox
# scc-srv
sudo bash scc-srv/scc-srv_gen_all 1.31.0-rsync-proxmox
```

# Configuration

## scc-srv rsync



```shell script
# /etc/rsyncd.conf

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

# Usage

### Updating scc

```shell script
/opt/scc-srv/bin/scc-update -f
```

### Adding a new realm

```shell script
/opt/scc-srv/bin/scc-realm -a $REALM_NAME
```

### Deleting a realm

```shell script
/opt/scc-srv/bin/scc-realm -d $REALM_NAME
```

### Adding hosts to realm

```shell script
scc-realm --add --quick --list $HOST_1 $REALM_NAME # do not call scc-update, manually call /opt/scc-srv/bin/scc-update -f
scc-realm --add --list $HOST_1,$HOST_2 $REALM_NAME
```

### Renaming a realm

```shell script
/opt/scc-srv/bin/scc-update -r $OLD_REALM_NAME $NEW_REALM_NAME
```

### Archiving hosts

```shell script
/opt/scc-srv/bin/scc-realm --archive /<dir>/archive --list $HOST_1,$HOST_2 --delete All
```

# Security

## Adding user:password to realm via .htpasswd

You can generate a password hash using https://hostingcanada.org/htpasswd-generator/

```shell script
# user1:123qwe
# user2:123qwe

sudo tee -a /var/opt/scc-srv/data/www/.htpasswd > /dev/null <<EOT
user1:{SHA}Bf50YcYHwzIpdy1AJQVgEBan0Oo=
user2:{SHA}Bf50YcYHwzIpdy1AJQVgEBan0Oo=
EOT
```

## Adding folder protection via .htaccess

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

## Adding group protection via .htgroups

Activating apache2 authz_groupfile

```shell script
a2enmod authz_groupfile
systemctl restart apache2
```

### Groups File:

```shell script
# /var/opt/scc-srv/data/www/.htgroups

$GROUP_NAME: $GROUP_USER_1 $GROUP_USER_2 $GROUP_USER_3

example:
admin: user1 user2
paythem: user_1 user_2
```
