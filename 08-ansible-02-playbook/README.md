# Домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению
1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.
2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соотвтествии с группами из предподготовленного playbook. 
4. Скачайте дистрибутив [java](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) и положите его в директорию `playbook/files/`. 

## Основная часть
1. Приготовьте свой собственный inventory файл `prod.yml`.
  
```shell
root@deb11-test50:~/playbook# cat inventory/prod.yml
```
```yaml
---
  elasticsearch:
    hosts:
      elastic-01:
        ansible_connection: docker
  kibana:
    hosts:
      kibana-01:
        ansible_connection: docker
```
  
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
  
```yaml
- name: Install Kibana
  hosts: kibana
  tasks:
    - name: Upload tar.gz Kibana from remote URL
      ansible.builtin.get_url:
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        mode: '0644'
        timeout: 60
        force: true
        validate_certs: false
      register: get_kibana
      until: get_kibana is succeeded
      tags:
        - kibana
    - name: Create directrory for Kibana ({{ kibana_home }})
      ansible.builtin.file:
        path: "{{ kibana_home }}"
        state: directory
        mode: '0755'
      tags: kibana
    - name: Extract Kibana in the installation directory
      become: true
      ansible.builtin.unarchive:
        copy: false
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "{{ kibana_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ kibana_home }}/bin/kibana"
      tags: kibana
    - name: Set environment Kibana
      become: true
      ansible.builtin.template:
        src: templates/kib.sh.j2
        dest: /etc/profile.d/kib.sh
        mode: '0755'
      tags: kibana
```
  

3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
  
```shell
root@deb11-test50:~/Ansible-Intro/playbook# ansible-lint site.yml
root@deb11-test50:~/Ansible-Intro/playbook#

```
  
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
  
```shell
ansible-playbook -i inventory/prod.yml site.yml --check
```

<details>
<summary>Вывод выполнения команды</summary>

```shell
root@deb11-test50:~/Ansible-Intro/playbook# ansible-playbook -i inventory/prod.yml site.yml --check

PLAY [Install Java] *************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host kibana-01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the
discovered platform python for this host. See https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be disabled
 by setting deprecation_warnings=False in ansible.cfg.
ok: [kibana-01]
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host elastic-01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the
discovered platform python for this host. See https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be disabled
 by setting deprecation_warnings=False in ansible.cfg.
ok: [elastic-01]

TASK [Set facts for Java DK vars] ***********************************************************************************************************************************************************************************************************
ok: [elastic-01]
ok: [kibana-01]

TASK [Upload .tar.gz file containing binaries from local storage] ***************************************************************************************************************************************************************************
changed: [elastic-01]
changed: [kibana-01]

TASK [Ensure installation dir exists] *******************************************************************************************************************************************************************************************************
changed: [elastic-01]
changed: [kibana-01]

TASK [Extract java in the installation directory] *******************************************************************************************************************************************************************************************
fatal: [elastic-01]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/17' must be an existing dir"}
fatal: [kibana-01]: FAILED! => {"changed": false, "msg": "dest '/opt/jdk/17' must be an existing dir"}

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
elastic-01                 : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
kibana-01                  : ok=4    changed=2    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0



```

</details>

7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
  
```shell
ansible-playbook -i inventory/prod.yml site.yml --diff
```

<details>
<summary>Вывод выполнения команды</summary>
  
```shell
root@deb11-test50:~/Ansible-Intro/playbook# ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] *************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host elastic-01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the
discovered platform python for this host. See https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be disabled
 by setting deprecation_warnings=False in ansible.cfg.
ok: [elastic-01]
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host kibana-01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the
discovered platform python for this host. See https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be disabled
 by setting deprecation_warnings=False in ansible.cfg.
ok: [kibana-01]

TASK [Set facts for Java DK vars] ***********************************************************************************************************************************************************************************************************
ok: [elastic-01]
ok: [kibana-01]

TASK [Upload .tar.gz file containing binaries from local storage] ***************************************************************************************************************************************************************************
diff skipped: source file size is greater than 104448
changed: [kibana-01]
diff skipped: source file size is greater than 104448
changed: [elastic-01]

TASK [Ensure installation dir exists] *******************************************************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/17",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elastic-01]
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/jdk/17",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana-01]

TASK [Extract java in the installation directory] *******************************************************************************************************************************************************************************************
changed: [elastic-01]
changed: [kibana-01]

TASK [Export environment variables] *********************************************************************************************************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-50898n9qfzk8o/tmp_r5w5fz0/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/17
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [kibana-01]
--- before
+++ after: /root/.ansible/tmp/ansible-local-50898n9qfzk8o/tmpf6jcsjpi/jdk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export JAVA_HOME=/opt/jdk/17
+export PATH=$PATH:$JAVA_HOME/bin
\ No newline at end of file

changed: [elastic-01]

PLAY [Install Elasticsearch] ****************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [elastic-01]

TASK [Upload tar.gz Elasticsearch from remote URL] ******************************************************************************************************************************************************************************************
changed: [elastic-01]

TASK [Create directrory for Elasticsearch] **************************************************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/elastic/8.3.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [elastic-01]

TASK [Extract Elasticsearch in the installation directory] **********************************************************************************************************************************************************************************
changed: [elastic-01]

TASK [Set environment Elastic] **************************************************************************************************************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-50898n9qfzk8o/tmpj6_ul4bc/elk.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export ES_HOME=/opt/elastic/8.3.1
+export PATH=$PATH:$ES_HOME/bin
\ No newline at end of file

changed: [elastic-01]

PLAY [Install Kibana] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [kibana-01]

TASK [Upload tar.gz Kibana from remote URL] *************************************************************************************************************************************************************************************************
changed: [kibana-01]

TASK [Create directrory for Kibana (/opt/kibana/8.3.1)] *************************************************************************************************************************************************************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/kibana/8.3.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [kibana-01]

TASK [Extract Kibana in the installation directory] *****************************************************************************************************************************************************************************************
changed: [kibana-01]

TASK [Set environment Kibana] ***************************************************************************************************************************************************************************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-50898n9qfzk8o/tmpqjnate00/kib.sh.j2
@@ -0,0 +1,5 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten on next playbook run.
+#!/usr/bin/env bash
+
+export KIBANA_HOME=/opt/kibana/8.3.1
+export PATH=$PATH:$KIBANA_HOME/bin

changed: [kibana-01]

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
elastic-01                 : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
kibana-01                  : ok=11   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

root@deb11-test50:~/Ansible-Intro/playbook#
```

</details>
  
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

```shell
ansible-playbook -i inventory/prod.yml site.yml --diff
```

<details>
<summary>Вывод выполнения команды</summary>
  
```shell
root@deb11-test50:~/Ansible-Intro/playbook# ansible-playbook -i inventory/prod.yml site.yml --diff

PLAY [Install Java] *************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host elastic-01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the
discovered platform python for this host. See https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be disabled
 by setting deprecation_warnings=False in ansible.cfg.
ok: [elastic-01]
[DEPRECATION WARNING]: Distribution Ubuntu 18.04 on host kibana-01 should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility with prior Ansible releases. A future Ansible release will default to using the
discovered platform python for this host. See https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. Deprecation warnings can be disabled
 by setting deprecation_warnings=False in ansible.cfg.
ok: [kibana-01]

TASK [Set facts for Java DK vars] ***********************************************************************************************************************************************************************************************************
ok: [elastic-01]
ok: [kibana-01]

TASK [Upload .tar.gz file containing binaries from local storage] ***************************************************************************************************************************************************************************
ok: [elastic-01]
ok: [kibana-01]

TASK [Ensure installation dir exists] *******************************************************************************************************************************************************************************************************
ok: [kibana-01]
ok: [elastic-01]

TASK [Extract java in the installation directory] *******************************************************************************************************************************************************************************************
skipping: [elastic-01]
skipping: [kibana-01]

TASK [Export environment variables] *********************************************************************************************************************************************************************************************************
ok: [elastic-01]
ok: [kibana-01]

PLAY [Install Elasticsearch] ****************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [elastic-01]

TASK [Upload tar.gz Elasticsearch from remote URL] ******************************************************************************************************************************************************************************************
ok: [elastic-01]

TASK [Create directrory for Elasticsearch] **************************************************************************************************************************************************************************************************
ok: [elastic-01]

TASK [Extract Elasticsearch in the installation directory] **********************************************************************************************************************************************************************************
skipping: [elastic-01]

TASK [Set environment Elastic] **************************************************************************************************************************************************************************************************************
ok: [elastic-01]

PLAY [Install Kibana] ***********************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [kibana-01]

TASK [Upload tar.gz Kibana from remote URL] *************************************************************************************************************************************************************************************************
ok: [kibana-01]

TASK [Create directrory for Kibana (/opt/kibana/8.3.1)] *************************************************************************************************************************************************************************************
ok: [kibana-01]

TASK [Extract Kibana in the installation directory] *****************************************************************************************************************************************************************************************
skipping: [kibana-01]

TASK [Set environment Kibana] ***************************************************************************************************************************************************************************************************************
ok: [kibana-01]

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
elastic-01                 : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
kibana-01                  : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

root@deb11-test50:~/Ansible-Intro/playbook#
```
  
</details>

9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
  
[README.md](https://github.com/OleKirs/Ansible-Intro/tree/master/playbook)
  
10. Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.
  
[https://github.com/OleKirs/Ansible-Intro.git](https://github.com/OleKirs/Ansible-Intro.git)
  

## Необязательная часть

1. Приготовьте дополнительный хост для установки logstash.
2. Пропишите данный хост в `prod.yml` в новую группу `logstash`.
3. Дополните playbook ещё одним play, который будет исполнять установку logstash только на выделенный для него хост.
4. Все переменные для нового play определите в отдельный файл `group_vars/logstash/vars.yml`.
5. Logstash конфиг должен конфигурироваться в части ссылки на elasticsearch (можно взять, например его IP из facts или определить через vars).
6. Дополните README.md, протестируйте playbook, выложите новую версию в github. В ответ предоставьте ссылку на репозиторий.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

