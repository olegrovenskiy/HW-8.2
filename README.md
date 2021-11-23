# HW-8.2

##  Playbook с описанием

Плейбук содержит три однотипных блока для Java, ElastickSearch and Kibana

1. Скачивание инсталяционного файла. java с локальной дирректории, Elastic, Kibana с Интернета
2. Загрузка инсталяционного файла в /tmp/
3. Разархивирование установочного ыайла в локальную директорию
	3.1 путь к директории прописан в template
	3.2 проводится проверка что инсталяционная директория сцществует
4. Каждый блок имеет свои теги: java, elastic, kibana
5. в group_vars прописаны соответствующие версии инсталяционных файлов и шаблон пути к загрузке
6. Загрузка проводится с проверкой усрешного выполнения
7. используется become для поднятия прав

	    vagrant@vagrant:/etc/ansible/HW-8.2$ cat site.yml
	    ---
	    - name: Install Java
	      hosts: all
	      tasks:
		- name: Set facts for Java 11 vars
		  set_fact:
		    java_home: "/opt/jdk/{{ java_jdk_version }}"
		  tags: java
		- name: Upload .tar.gz file containing binaries from local storage
		  copy:
		    src: "{{ java_oracle_jdk_package }}"
		    dest: "/tmp/jdk-{{ java_jdk_version }}.tar.gz"
		  register: download_java_binaries
		  until: download_java_binaries is succeeded
		  tags: java
		- name: Ensure installation dir exists
		  become: true
		  file:
		    state: directory
		    path: "{{ java_home }}"
		  tags: java
		- name: Extract java in the installation directory
		  become: true
		  unarchive:
		    copy: false
		    src: "/tmp/jdk-{{ java_jdk_version }}.tar.gz"
		    dest: "{{ java_home }}"
		    extra_opts: [--strip-components=1]
		    creates: "{{ java_home }}/bin/java"
		  tags:
		    - java
		- name: Export environment variables
		  become: true
		  template:
		    src: jdk.sh.j2
		    dest: /etc/profile.d/jdk.sh
		  tags: java
	    - name: Install Elasticsearch
	      hosts: elasticsearch
	      tasks:
		- name: Upload tar.gz Elasticsearch from remote URL
		  get_url:
		    url: "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz"
		    dest: "/tmp/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz"
		    mode: 0755
		    timeout: 60
		    force: true
		    validate_certs: false
		  register: get_elastic
		  until: get_elastic is succeeded
		  tags: elastic
		- name: Create directrory for Elasticsearch
		  file:
		    state: directory
		    path: "{{ elastic_home }}"
		  tags: elastic
		- name: Extract Elasticsearch in the installation directory
		  become: true
		  unarchive:
		    copy: false
		    src: "/tmp/elasticsearch-{{ elastic_version }}-linux-x86_64.tar.gz"
		    dest: "{{ elastic_home }}"
		    extra_opts: [--strip-components=1]
		    creates: "{{ elastic_home }}/bin/elasticsearch"
		  tags:
		    - elastic
		- name: Set environment Elastic
		  become: true
		  template:
		    src: templates/elk.sh.j2
		    dest: /etc/profile.d/elk.sh
		  tags: elastic
	    - name: Install Kibana
	      hosts: kibana
	      tasks:
		- name: Upload tar.gz Kibana from remote URL
		  get_url:
		    url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
		    dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
		    mode: 0755
		    timeout: 60
		    force: true
		    validate_certs: false
		  register: get_kibana
		  until: get_kibana is succeeded
		  tags: kibana
		- name: Create directrory for Kibana
		  file:
		    state: directory
		    path: "{{ kibana_home }}"
		  tags: kibana
		- name: Extract kibana in the installation directory
		  become: true
		  unarchive:
		    copy: false
		    src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
		    dest: "{{ kibana_home }}"
		    extra_opts: [--strip-components=1]
		    creates: "{{ kibana_home }}/bin/kibana"
		  tags:
		    - kibana
		- name: Set environment Kibana
		  become: true
		  template:
		    src: templates/kibana.sh.j2
		    dest: /etc/profile.d/kibana.sh
		  tags: kibana

	    vagrant@vagrant:/etc/ansible/HW-8.2$

---------------------------


Результаты работы:

Подготовка:

1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.

https://github.com/olegrovenskiy/HW-8.2

2. Скачайте playbook из репозитория с домашним заданием и перенесите его в свой репозиторий.

скачено

Подготовьте хосты в соотвтествии с группами из предподготовленного playbook.

localhost

все хосты - local. elastic, kibana будут использовать localhost

3. Скачайте дистрибутив java и положите его в директорию playbook/files/.

    vagrant@vagrant:/etc/ansible/HW-8.2/files$ ls -l
    total 8

    jdk-11.0.13_linux-x64_bin.tar.gz

файл скачен с Оракл и через WinSCP загружен на ВМ в ОраксБокс с Ансибл

при загрузке в рнпозиторий файл был удалён, из за большого размера

Основная часть

1. Приготовьте свой собственный inventory файл prod.yml.

все хосты используют localhostм подключением local

    ---
    elasticsearch:
      hosts:
        localhost:
          ansible_connection: local
    local:
        hosts:
          localhost:
             ansible_connection: local
    kibana:
        hosts:
          localhosts:
             ansible_connection: local

2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.

запуск исходного playbook после скачивания java, корректировки версии

    vagrant@vagrant:/etc/ansible/HW-8.2$ sudo ansible-playbook site.yml -i inventory/prod.yml

    PLAY [Install Java] ***********************************************************************************************************************

    TASK [Gathering Facts] ********************************************************************************************************************
    ok: [localhost]

    TASK [Set facts for Java 11 vars] *********************************************************************************************************
    ok: [localhost]

    TASK [Upload .tar.gz file containing binaries from local storage] *************************************************************************
    changed: [localhost]

    TASK [Ensure installation dir exists] *****************************************************************************************************
    ok: [localhost]

    TASK [Extract java in the installation directory] *****************************************************************************************
    changed: [localhost]

    TASK [Export environment variables] *******************************************************************************************************
    changed: [localhost]

    PLAY [Install Elasticsearch] **************************************************************************************************************

    TASK [Gathering Facts] ********************************************************************************************************************
    ok: [localhost]

    TASK [Upload tar.gz Elasticsearch from remote URL] ****************************************************************************************
    changed: [localhost]

    TASK [Create directrory for Elasticsearch] ************************************************************************************************
    changed: [localhost]

    TASK [Extract Elasticsearch in the installation directory] ********************************************************************************
    changed: [localhost]

    TASK [Set environment Elastic] ************************************************************************************************************
    changed: [localhost]

    PLAY RECAP ********************************************************************************************************************************
    localhost                  : ok=11   changed=7    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

    vagrant@vagrant:/etc/ansible/HW-8.2$


-------------------

добавил host kibana в инвентори

    vagrant@vagrant:/etc/ansible/HW-8.2/inventory$ cat prod.yml
    ---
    elasticsearch:
      hosts:
        localhost:
          ansible_connection: local
    local:
        hosts:
          localhost:
             ansible_connection: local
    kibana:
        hosts:
          localhosts:
             ansible_connection: local

    vagrant@vagrant:/etc/ansible/HW-8.2/inventory$

    ----

Переменные для Кибана

    vagrant@vagrant:/etc/ansible/HW-8.2/group_vars/kibana$ cat vars.yml
    ---
    kibana_version: "7.15.2"
    kibana_home: "/opt/kibana/{{ kibana_version }}"
    vagrant@vagrant:/etc/ansible/HW-8.2/group_vars/kibana$

    ------

Новый таск

    - name: Install Kibana
      hosts: kibana
      tasks:
        - name: Upload tar.gz Kibana from remote URL
          get_url:
            url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
            dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
            mode: 0755
            timeout: 60
            force: true
            validate_certs: false
          register: get_kibana
          until: get_kibana is succeeded
          tags: kibana
        - name: Create directrory for Kibana
          file:
            state: directory
            path: "{{ kibana_home }}"
          tags: kibana
        - name: Extract kibana in the installation directory
          become: true
          unarchive:
            copy: false
            src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
            dest: "{{ kibana_home }}"
            extra_opts: [--strip-components=1]
            creates: "{{ kibana_home }}/bin/kibana"
          tags:
            - kibana
        - name: Set environment Kibana
          become: true
          template:
            src: templates/kibana.sh.j2
            dest: /etc/profile.d/kibana.sh
          tags: kibana


    -----------------------

3. При создании tasks рекомендую использовать модули: get_url, template, unarchive, file.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.

запуск плейбук и результат

    vagrant@vagrant:/etc/ansible/HW-8.2$ sudo ansible-playbook site.yml -i inventory/prod.yml

    PLAY [Install Java] ***********************************************************************************************************************

    TASK [Gathering Facts] ********************************************************************************************************************
    ok: [localhost]
    ok: [localhosts]

    TASK [Set facts for Java 11 vars] *********************************************************************************************************
    ok: [localhost]
    ok: [localhosts]

    TASK [Upload .tar.gz file containing binaries from local storage] *************************************************************************
    ok: [localhost]
    ok: [localhosts]

    TASK [Ensure installation dir exists] *****************************************************************************************************
    ok: [localhosts]
    ok: [localhost]

    TASK [Extract java in the installation directory] *****************************************************************************************
    skipping: [localhost]
    skipping: [localhosts]

    TASK [Export environment variables] *******************************************************************************************************
    ok: [localhost]
    ok: [localhosts]

    PLAY [Install Elasticsearch] **************************************************************************************************************

    TASK [Gathering Facts] ********************************************************************************************************************
    ok: [localhost]

    TASK [Upload tar.gz Elasticsearch from remote URL] ****************************************************************************************
    ok: [localhost]

    TASK [Create directrory for Elasticsearch] ************************************************************************************************
    ok: [localhost]

    TASK [Extract Elasticsearch in the installation directory] ********************************************************************************
    skipping: [localhost]

    TASK [Set environment Elastic] ************************************************************************************************************
    ok: [localhost]

    PLAY [Install Kibana] *********************************************************************************************************************

    TASK [Gathering Facts] ********************************************************************************************************************
    ok: [localhosts]

    TASK [Upload tar.gz Kibana from remote URL] ***********************************************************************************************
    changed: [localhosts]

    TASK [Create directrory for Kibana] *******************************************************************************************************
    changed: [localhosts]

    TASK [Extract kibana in the installation directory] ***************************************************************************************
    changed: [localhosts]

    TASK [Set environment Kibana] *************************************************************************************************************
    changed: [localhosts]

    PLAY RECAP ********************************************************************************************************************************
    localhost                  : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
    localhosts                 : ok=10   changed=4    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0



    5. Запустите ansible-lint site.yml и исправьте ошибки, если они есть.

    есть не критичные ворнинги

    vagrant@vagrant:/etc/ansible/HW-8.2$ sudo ansible-lint site.yml
    WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
    WARNING  Listing 9 violation(s) that are fatal
    risky-file-permissions: File permissions unset or incorrect
    site.yml:9 Task/Handler: Upload .tar.gz file containing binaries from local storage

    risky-file-permissions: File permissions unset or incorrect
    site.yml:16 Task/Handler: Ensure installation dir exists

    risky-file-permissions: File permissions unset or incorrect
    site.yml:32 Task/Handler: Export environment variables

    risky-file-permissions: File permissions unset or incorrect
    site.yml:52 Task/Handler: Create directrory for Elasticsearch

    risky-file-permissions: File permissions unset or incorrect
    site.yml:67 Task/Handler: Set environment Elastic

    yaml: too many blank lines (3 > 2) (empty-lines)
    site.yml:75

    risky-file-permissions: File permissions unset or incorrect
    site.yml:90 Task/Handler: Create directrory for Kibana

    risky-file-permissions: File permissions unset or incorrect
    site.yml:105 Task/Handler: Set environment Kibana

    yaml: too many blank lines (1 > 0) (empty-lines)
    site.yml:111

    You can skip specific rules or tags by adding them to your configuration file:
    # .ansible-lint
    warn_list:  # or 'skip_list' to silence them completely
      - experimental  # all rules tagged as experimental
      - yaml  # Violations reported by yamllint

    Finished with 2 failure(s), 7 warning(s) on 1 files.
    vagrant@vagrant:/etc/ansible/HW-8.2$





