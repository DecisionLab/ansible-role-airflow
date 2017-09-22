# airflow

[![Build Status](https://travis-ci.org/infOpen/ansible-role-airflow.svg?branch=master)](https://travis-ci.org/infOpen/ansible-role-airflow)

Install airflow package.

## Requirements

This role requires Ansible 2.0 or higher,
and platform requirements are listed in the metadata file.

## Testing

This role use [Molecule](https://github.com/metacloud/molecule/) to run tests.

Locally, you can run tests on Docker (default driver) or Vagrant.
Travis run tests using Docker driver only.

Currently, tests are done on:
- Debian Jessie
- Ubuntu Trusty
- Ubuntu Xenial

and use:
- Ansible 2.0.x
- Ansible 2.1.x
- Ansible 2.2.x
- Ansible 2.3.x

### Running tests

#### Using Docker driver

```
$ tox
```

#### Using Vagrant driver

```
$ MOLECULE_DRIVER=vagrant tox
```

## Role Variables

### Default role variables

``` yaml
# Installation vars
airflow_user_name: 'airflow'
airflow_user_group: "{{ airflow_user_name }}"
airflow_user_shell: '/bin/false'
airflow_user_home_path: '/var/lib/airflow'
airflow_user_home_mode: '0700'

airflow_log_path: '/var/log/airflow'
airflow_log_owner: "{{ airflow_user_name }}"
airflow_log_group: "{{ airflow_user_group }}"
airflow_log_mode: '0700'

airflow_pid_path: '/var/run/airflow'
airflow_pid_owner: "{{ airflow_user_name }}"
airflow_pid_group: "{{ airflow_user_group }}"
airflow_pid_mode: '0700'

airflow_virtualenv: "{{ airflow_user_home_path }}/venv"
airflow_python_version: 'python3.4'
airflow_packages:
  - name: 'Cython'
    version: '0.24'
  - name: 'airflow'
    version: '1.7.1.3'
  - name: 'setuptools'
    version: '23.0.0'
airflow_extra_packages:
  - name: 'airflow[crypto]'
airflow_system_dependencies: []

# Services management
airflow_managed_upstart_services: []
airflow_managed_initd_services:
  - 'airflow-scheduler'
  - 'airflow-webserver'

airflow_services_states:
  - name: 'airflow-webserver'
    enabled: True
    state: 'started'
  - name: 'airflow-scheduler'
    enabled: True
    state: 'started'


# Webserver service configuration
airflow_webserver_port: 8080
airflow_webserver_workers: 1
airflow_webserver_worker_class: 'sync'
airflow_webserver_worker_timeout: 30
airflow_webserver_hostname: "{{ ansible_default_ipv4.address }}"
airflow_webserver_pid_file: "{{ airflow_pid_path }}/airflow-webserver.pid"
airflow_webserver_log_file: "{{ airflow_log_path }}/airflow-webserver.log"
airflow_webserver_debug: False
airflow_webserver_respawn_limit_count: 5
airflow_webserver_respawn_limit_timeperiod: 30

# Scheduler service configuration
airflow_scheduler_runs: 0
airflow_scheduler_pid_file: "{{ airflow_pid_path }}/airflow-scheduler.pid"
airflow_scheduler_log_file: "{{ airflow_log_path }}/airflow-scheduler.log"
airflow_scheduler_respawn_limit_count: 5
airflow_scheduler_respawn_limit_timeperiod: 30

# Databases variables
airflow_manage_database: True
airflow_database_engine: 'mysql'

# Set do_init to false if database already initialized
airflow_do_init_db: True
airflow_do_upgrade_db: True

# Default configuration
airflow_defaults_config:
  core:
    home: "{{ airflow_user_home_path ~ '/airflow' }}"
    dags_folder: "{{ airflow_user_home_path ~ '/airflow/dags' }}"
    base_log_folder: "{{ airflow_log_path }}"
    remote_base_log_folder: ''
    remote_log_conn_id: ''
    encrypt_s3_logs: False
    executor: 'SequentialExecutor'
    sql_alchemy_conn: 'sqlite:////var/lib/airflow/airflow/airflow.db'
    sql_alchemy_pool_size: 5
    sql_alchemy_pool_recycle: 3600
    parallelism: 32
    dag_concurrency: 16
    dags_are_paused_at_creation: True
    non_pooled_task_slot_count: 128
    max_active_runs_per_dag: 16
    load_examples: False
    plugins_folder: "{{ airflow_user_home_path ~ '/airflow/plugins' }}"
    fernet_key: 'cryptography_not_found_storing_passwords_in_plain_text'
    donot_pickle: False
    dagbag_import_timeout: 30

  operators:
    default_owner: 'Airflow'

  webserver:
    base_url: 'http://localhost:8080'
    web_server_host: '0.0.0.0'
    web_server_port: 8080
    web_server_worker_timeout: 120
    secret_key: 'temporary_key'
    workers: 4
    worker_class: sync
    expose_config: True
    authenticate: False
    filter_by_owner: False

  email:
    email_backend: 'airflow.utils.email.send_email_smtp'

  smtp:
    smtp_host: 'localhost'
    smtp_starttls: True
    smtp_ssl: False
    smtp_user: 'airflow'
    smtp_port: 25
    smtp_password: 'airflow'
    smtp_mail_from: 'airflow@airflow.com'

  celery:
    celery_app_name: 'airflow.executors.celery_executor'
    celeryd_concurrency: 16
    worker_log_server_port: 8793
    broker_url: 'sqla+mysql://airflow:airflow@localhost:3306/airflow'
    celery_result_backend: 'db+mysql://airflow:airflow@localhost:3306/airflow'
    flower_port: 5555
    default_queue: 'default'

  scheduler:
    job_heartbeat_sec: 5
    scheduler_heartbeat_sec: 5
    statsd_on: False
    statsd_host: 'localhost'
    statsd_port: 8125
    statsd_prefix: 'airflow'
    max_threads: 2

  mesos:
    master: 'localhost:5050'
    framework_name: 'Airflow'
    task_cpu: 1
    task_memory: 256
    checkpoint: False
    failover_timeout: 604800
    authenticate: False
    default_principal: 'admin'
    default_secret: 'admin'

airflow_user_config: {}
airflow_config: "{{ airflow_defaults_config | combine(airflow_user_config) }}"

# Logrotate configuration
airflow_logrotate_config:
  - filename: '/etc/logrotate.d/airflow'
    log_pattern:
      - "{{ airflow_log_path }}/*.log"
    options:
      - 'rotate 12'
      - 'weekly'
      - 'compress'
      - 'delaycompress'
      - "create 640 {{ airflow_log_owner }} {{ airflow_log_group }}"
      - 'postrotate'
      - 'endscript'
```

## Dependencies

None

## Example Playbook

``` yaml
- hosts: servers
  roles:
    - { role: infOpen.airflow }
```

## License

MIT

## Author Information

Alexandre Chaussier (for Infopen company)
- http://www.infopen.pro
- a.chaussier [at] infopen.pro
