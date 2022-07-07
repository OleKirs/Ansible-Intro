# Playbook 'site.yml'

## Структура файлов и каталогов Playbook
```
.
├── docker-compose.yml
├── files
│   └── jdk-17_linux-x64_bin.tar.gz
├── group_vars
│   ├── all
│   │   └── vars.yml
│   ├── elasticsearch
│   │   └── vars.yml
│   └── kibana
│       └── vars.yml
├── inventory
│   └── prod.yml
├── site.yml
└── templates
    ├── elk.sh.j2
    ├── jdk.sh.j2
    └── kib.sh.j2
```

## Назначение
Предназначен для автоматизации установки программного обеспечения (ПО) Elasticsearch и Kibana внутри контейнеров Docker при помощи ПО Ansible.

## Предварительные условия
Перед запуском Playbook необходимо:
1. Скачать с сайта [Java](https://www.oracle.com/java/technologies/downloads/) актуальную версию дистрибутива JDK в формате `tar.gz` и поместить его в директорию `./files` внутри репозитория.
2. Проверить и при необходимости изменить значения переменных в файле `group_vars/all/vars.yml`
3. Определить актуальные версии ПО Elasticsearch и Kibana. Актуализировать значения переменных в файлах `group_vars/elasticsearch/vars.yml` и `group_vars/elasticsearch/vars.yml`
4. Актуализировать имена хостов (docker-контейнеров) для размещения ПО и способы подключения в файле `inventory/prod.yml`
5. Запустить работу docker-контейнеров, предназначенных для установки ПО. 
6. Предоставить docker-контейнерам, предназначенным для установки ПО, доступ к сайтам с дистрибутивами ПО Elasticsearch и Kibana. 

## Описание тэгов в Playbook
* Тег `java` связан с задачами предварительной установки в контейнеры ПО JDK
* Тег `elastic` связан с задачами получения и установки ПО Elasticsearch
* Тег `kibana` связан с задачами получения и установки ПО Kibana

## Групповые переменные
Расположены в файлах `vars.yml` в соответствующих подкаталогах в каталоге `group_vars`

1. `group_vars/all/vars.yml`:  
1.1. `java_jdk_version` - версия ПО JDK скачанного и расположенного в каталоге `files\`    
1.2. `java_oracle_jdk_package` - имя файла дистрибутива JDK в каталоге `files\`  
2. `group_vars/elasticsearch/vars.yml`  
2.1. `elastic_version` - версия ПО Elasticsearch для скачивания и установки  
2.2. `elastic_home`- вычисляемая переменная, каталог  для установки ПО внутри контейнера
3. `group_vars/kibana/vars.yml`  
3.1. `kibana_version` - версия ПО Kibana для скачивания и установки  
3.2. `kibana_home` - вычисляемая переменная, каталог  для установки ПО внутри контейнера  

### Пример запуска Playbook

```shell
ansible-playbook -i inventory/prod.yml site.yml
```

