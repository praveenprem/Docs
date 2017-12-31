# **__MySQL installation on Ubuntu 16.04__**

---

## Table of content

- [Installation](#install)
- [Security upgrade](#security)
- [Create user](#account)
- [User Permissions](#grantPermission)
    - [Grant](#grant)
    - [Revoke](#revoke)
    - [Reload privileges](#releadPrivileges)
- [Remove user](#deleteAccount)

---

<a name="install"/>

### Installation
```
apt -y update && apt -y upgrade
apt - install mysql-server
```

<a name="security"/>

### Security script

Run this to upgrade the server security

```
mysql_secure_installation
```

<a name="account"/>

### Create user
```
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
```

<a name="grantPermission"/>

### User Permissions
Available permissions
  - `ALL PRIVILEGES` - as we saw previously, this would allow a MySQL user all access to a designated database (or if no database is selected, across the system)
  - `CREATE` - allows them to create new tables or databases
  - `DROP` - allows them to them to delete tables or databases
  - `DELETE` - allows them to delete rows from tables
  - `INSERT` - allows them to insert rows into tables
  - `SELECT` - allows them to use the Select command to read through databases
  - `UPDATE` - allow them to update table rows
  - `GRANT OPTION` - allows them to grant or remove other users' privileges

<a name="grant"/>

###### Grant

Permission grant syntax
```
GRANT [type of permission] ON [database name].[table name] TO ‘[username]’@'localhost’;
GRANT ALL PRIVILEGES ON * . * TO 'newuser'@'localhost';
```

<a name="revoke"/>

###### Revoke

Permission revoke syntax
```
REVOKE [type of permission] ON [database name].[table name] FROM ‘[username]’@‘localhost’;
```

<a name="releadPrivileges"/>

###### Reload privileges

```
FLUSH PRIVILEGES;
```
<a name="deleteAccount"/>

### Remove user

Drop user syntax
```
DROP USER ‘[username]’@‘localhost’;
```
