# Роль postgres

Роль устанавливает PostgreSQL на Ubuntu и выполняет его основную настройку.

Я делал её так, чтобы основные параметры можно было менять через переменные, а не переписывать задачи Ansible.

## Что делает роль

Сначала устанавливается PostgreSQL из стандартных репозиториев Ubuntu. Затем создаётся каталог для данных и выполняется инициализация через pg_ctl initdb.

По умолчанию данные хранятся в /data/postgres/data.

После этого настраиваются postgresql.conf и pg_hba.conf. Параметры первого файла передаются словарём, а правила авторизации — списком словарей.

PostgreSQL запускается через собственную systemd-службу postgres-custom.service. При изменении конфигурации используется хендлер перезапуска.

Роль также создаёт пользователей и базы данных из списков переменных.

В конце устанавливается postgres-exporter, который собирает метрики PostgreSQL. Для него создаётся отдельная systemd-служба.

## Переменные

Все основные настройки находятся в defaults/main.yml.

| Переменная | Назначение |
|---|---|
| postgres__version | Версия PostgreSQL, по умолчанию 16 |
| postgres__data_dir | Каталог данных, по умолчанию /data/postgres/data |
| postgres__opts | Параметры postgresql.conf |
| postgres__hba_rules | Правила авторизации pg_hba.conf |
| postgres__users | Список пользователей |
| postgres__databases | Список баз данных |
| postgres__exporter_version | Версия экспортера |
| postgres__exporter_port | Порт экспортера, по умолчанию 9187 |
| postgres__exporter_user | Пользователь для запуска экспортера |
| postgres__exporter_sha256 | Контрольная сумма архива экспортера |

Внутренние переменные находятся в vars/main.yml и начинаются с двух подчёркиваний (__).

## Пример плейбука

```yaml
---
- name: Install PostgreSQL on vm1
  hosts: vm1
  become: true
  roles:
    - postgres
```

Пример настройки переменных:

```yaml
postgres__data_dir: /data/postgres/data

postgres__opts:
  listen_addresses: "'127.0.0.1'"
  port: 5432
  max_connections: 100

postgres__users:
  - name: app_user
    password: "{{ vault__app_user_password }}"
    role_attr_flags: LOGIN

postgres__databases:
  - name: app_db
    owner: app_user
```

Пароль пользователя хранится в Ansible Vault, чтобы не оставлять его открытым в репозитории.

## Идемпотентность

Чтобы повторный запуск не вносил лишних изменений, я использовал обычные модули Ansible и проверку creates для initdb.

Например, если файл PG_VERSION уже существует, кластер не инициализируется повторно.

Конфигурационные файлы обновляются только при изменении параметров. Перезапуск служб происходит через хендлеры, а не при каждом запуске.

Загрузка и распаковка экспортера тоже не выполняются повторно без необходимости.

## Проверка работы

Проверка PostgreSQL:

    systemctl is-active postgres-custom.service

    sudo -u postgres psql -c 'SELECT 1'

Проверка экспортера:

    systemctl is-active postgres-exporter.service

    curl -s http://127.0.0.1:9187/metrics | grep '^pg_up '

При успешном подключении экспортера к PostgreSQL получаем pg_up 1.

При повторном запуске плейбука без изменения настроек получено changed=0.
