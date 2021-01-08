# AJM - Ansible Jira Manager

Ansible playbook for install, backup and upgrade Jira instances on Windows or Linux servers (RHEL, CentOS, Debian, Ubuntu).

---

## Table of content

1. [Overview](#markdown-header-overview)
2. [Quickstart](#markdown-header-quickstart)
3. [Requirements](#markdown-header-requirements)
   1. [UNIX Ansible master](#markdown-header-unix-ansible-master)
      1. [Windows Hosts](#markdown-header-windows-hosts)
      2. [Linux Hosts](#markdown-header-linux-hosts)
   2. [Windows host Requirements](#markdown-header-windows-host-requirements)
      1. [Extend Windows file path length](#markdown-header-extend-windows-file-path-length)
      2. [Modules and software requirements](#markdown-header-modules-and-software-requirements)
4. [Usage](#markdown-header-usage)
   1. [Variables](#markdown-header-variables)
   2. [Windows or UNIX](#markdown-header-windows-or-unix)
   3. [Playbook arguments](#markdown-header-playbook-arguments)
5. [License](#markdown-header-license)

---

### Overview

Ansible playbook to automate install, upgrade and backup of Atlassian Jira.

The performed tasks are:

* create all directory needed for the procedure if not present into the system
* download of the standalone archive of a given version
* extracte the archive
* Create the user and/or group needed for Jira
* create the files:
  * jira-applications.properties
  * response.varfile
  * all files required for an upgrade (cacert, xml config files, template, ...)
* install, upgrade or backup

---

### Quickstart

```bash
# Use ansible galaxy to install the collection
$ ansible-galaxy install ajm
# Deploy to a remote
$ ansible-playbook ajm.yml --extra-vars=/path/to/file_vars
# or
$ ansible-playbook ajm.yml --extra-vars "ajm_action=[ACTION]"
```

---

### Requirements

In order to procede with an installation of a Jira tool is mandatory for the host(s) to have either JDK or OpenJDK present on the system and set as an environment variable.

The playbook take care to verify the presence of these requirements and to install JDK (OpenJDK version 8) and to set the environment variable if necessary. It is also possible to deactivate these steps for a Windows host by changing the variable `install_jdk` into the file `/roles/jira.inup_win/defaults/main.yml`

If you want to set manually the JDK (or JRE) remember that if the folder it is stored under _Program Files_ the variable path must be **C:\Progra~1\<PATH TO JDK>**, if it is under Program Files(x86) the path must be **C:\Progra~2\<PATH TO JDK>**


| Jira version | Oracle JRE/JDK version | OpenJDK version |
| - | - | - |
| 8.2 to 8.13 | 8 or 11 | 8 or 11 |
| 7.13 to 8.1 | 8 | 8 |
| 7.4 to 7.12 | 8 |   |

#### UNIX Ansible master

##### Windows Hosts

Example of the `hosts` file in `/etc/ansible/hosts` for a Windows host

```bash
[jiraservers]
1.2.3.4
1.2.3.5
1.2.3.6


[jiraservers:vars]
ansible_user=Administrator
ansible_password=PASSWORD
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

##### Linux Hosts

Example of the `hosts` file in `/etc/ansible/hosts` for a Linux host

```bash
[jiraservers]
1.2.3.4
1.2.3.5
1.2.3.6

[jiraservers:vars]
ansible_connection=ssh
ansible_user=<Ansible user>
ansible_password=<Password>
ansible_become=yes
ansible_become_method=sudo
ansible_become_password=<Password>
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

#### Windows host Requirements

##### Extend Windows file path length

Windows supports file paths up to 260 characters by default. This could be a problem during the extraction of the Jira standallone archive so we must to configure the system registry to support long file paths:

Verify or edit the Registry settings to support long paths:

* Open the Registry Editor: press Windows key and type **regedit** and press Enter key
* Navigate to **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem**
* Change **LongPathsEnabled** value to **1**. If you do not see LongPathsEnabled listed, you must create the entry by right-clicking the FileSystem key, chosing New > DWORD (32-bit) Value, and then naming the new value LongPathsEnabled.

Verify or edit any Group Policy to support long paths:

* Open Group Policy Editor: press Windows Key and type **gpedit.msc** and hit Enter key
* Navigate to the following directory: Local Computer Policy > Computer Configuration > Administrative Templates > System > Filesystem > NTFS.
* Click Enable NTFS long paths option and enable it.

##### Modules and software requirements

* Python 3
* winrm
* sqlcmd
* Pscx

---

### Usage

The tree of the playbook look like:

```python
ajm
  \_group_vars
    \_all
      \_vars_file.yml
  \_roles
    \_jira.inup_win
      \_defaults
        \_main.yml
      \_files
        \_jira-config.properties
        \_ ...
      \_handlers
        \_main.yml
      \_tasks
        \_backup.yml
        \_database.yml
        \_dependencies.yml
        \_install.yml
        \_main.yml
        \_needed_folders.yml
        \_start_jira.yml
        \_stop_jira.yml
        \_upgrade.yml
      \_templates
        \_jira-application.properties.j2
        \_mssql-dbconfig.xml.j2
        \_postgresql-dbconfig.xml.j2
        \_response.varfile.j2
        \_ ...
      \_tests
      \_vars
    \_jira.inup_linux
      \_ SAME AS ABOVE
  \_ajm.yml
```

#### Variables

Edit the **group_vars/all/vars_file.yml** in order to set all global variables (accessible and used from all roles).

```yaml
---
jira_user_group: jira

jira_lang: en
jira_rmi: 8005
jira_http_port: 8080

# variables used to chose the type and version of the Jira application
jira_type: core
jira_version: 8.6.0
jira_arc: x64

# Use this var list to add properties into the jira-config.properties file
# when the action is UPGRADE and the NO_FILE_CREATION is equal to False
jira_config_properties: 
- jira.export.excel.enabled=true

# The supported DB are MsSQL and PostgreSQL
# To set one use:
#   - mssql
#   - postgresql
# Empty to use the default H2 database
db_type:
db_port: 59890
db_name: DB_name
db_server: DB_SERVER
#MS SQL
db_host_mssql: jdbc:sqlserver://{{ db_server }}:{{ db_port }};databaseName={{ db_name }}
db_driver_class_mssql: com.microsoft.sqlserver.jdbc.SQLServerDriver
#PostgreSQL
db_host_postgresql: jdbc:postgresql://{{ db_server }}:{{ db_port }}/{{ db_name }}
db_driver_class_postgresql: org.postgresql.Driver

db_user: jiradbuser
db_pass: password

jira_user: jira
jira_group: jira
# created with:
# python -c 'import crypt; print crypt.crypt("This is my Password", "$1$SomeSalt$")'
jira_user_pwd: $1$SomeSalt$UqddPX3r4kH3UL5jq5/ZI

# Possible actions could be:
# - install
# - upgrade
# - backup
ajm_action: install

# if this var is equal to True a backup
# will be performed before to upgrade Jira
ajm_backup: True

# vars to enable the copy of:
# - email template folder
# - cacert folder
ajm_email_template_copy: False
ajm_cacert_copy: False

```

Both roles have another file for the variables:

* **roles/jira.inup_win/defaults/main.yml**
* **roles/jira.inup_linux/defaults/main.yml**

#### Windows or UNIX

The playbook will automatically use the correct role (Windows or UNIX).

* To backup:

  ```bash
  root@ansible-master:˜/ajm$ ansible-playbook ajm.yml --extra-vars "ajm_action=backup"
  ```

* To install:

  ```bash
  root@ansible-master:˜/ajm$ ansible-playbook ajm.yml
  ```

* To upgrade:

  ```bash
  root@ansible-master:˜/ajm$ ansible-playbook ajm.yml --extra-vars "ajm_action=upgrade"
  ```

* To upgrade without backup:

  ```bash
  root@ansible-master:˜/ajm$ ansible-playbook ajm.yml --extra-vars "ajm_action=upgrade ajm_backup=False"
  ```

#### Playbook arguments

* `jira_home_linux` / `jira_home_win`: JIRA HOME directory of the Atlassian software
* `jira_install_linux` / `jira_install_win`: directory of the current Jira version
* `jira_install_abs_linux` / `jira_install_abs_win`: root folder of the current Jira version, where the archive will be extracted and the install/upgrade procedure started
* `jira_tmp_linux` / `jira_tmp_win`: temporary directory
* `jira_bkp_linux` / `jira_bkp_win`: directory where the backups will be stored
* `jira_type`: could be either *Core* or *Software*
* `jira_version`: desired version of the Atlassian Jira (ex.: 8.5.4)
* `jira_config_properties`: list of configuration entries to be recorded into the *jira-applications.properties* file
* `ajm_action`: one of *install*, *backup* or *upgrade*. Default to *install*
* `ajm_backup`: could be *True* or *False* and it is used with upgrade action. Default to *False*
* `ajm_files_edit`: could be *True* or *False* and it is used with upgrade action. Default to *True*
* `ajm_email_template_copy`: could be *True* or *False* and it is used with upgrade action. Default to *False*
* `ajm_cacert_copy`: could be *True* or *False* and it is used with upgrade action. Default to *False*

---

### Author Information

Stefano Galati

galatistefano83@gmail.com