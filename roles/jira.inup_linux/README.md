jira.inup_linux

This is the role for installation and upgrade procedures in Windows hosts. (Debian, Ubuntu, CentOS, RHEL).

Requirements
------------

Any pre-requisite is demanded for this role.

Role Variables
--------------

All role variables are setted into `/defaults/main.yml` file

```yaml
---
jira_home_linux: '/var/atlassian/application-data/jira'
jira_install_linux: /opt/atlassian/jira/JIRA_8.7.0
jira_install_abs_linux: /opt/atlassian/jira
jira_bkp_linux: '/JIRA/JIRA_BACKUP'
jira_tmp_linux: '/tmp'
jira_exec_linux: atlassian-jira-{{ jira_type }}-{{ jira_version }}-{{ jira_arc }}.bin
jira_download_linux: https://www.atlassian.com/software/jira/downloads/binary/{{ jira_exec_linux }}
```