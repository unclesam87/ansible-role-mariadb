# Ansible role to install and configure MariaDB on Debian systems

[![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-mariadb-blue.svg)](https://galaxy.ansible.com/systemli/mariadb/) [![Build Status](https://travis-ci.org/systemli/ansible-role-mariadb.svg?branch=master)](https://travis-ci.org/systemli/ansible-role-mariadb)

The role installs and configures the MariaDB server on a Debian system.

* Installs `automysqlbackup` per default
* Sets `innodb_buffer_pool_instances` to number of vCPUs
* Sets `innodb_buffer_pool_size` to (total memory / 2)

The role is tested on Debian Jessie, Stretch, and Buster.

## Variables and their defaults

```
# MariaDB databases
mariadb_databases: []

# Additional MariaDB users
#mariadb_users:
#  - name: project
#    host: localhost
#    password: insecure
#    priv: "project.*:All"
mariadb_users: []

# Install automysqlbackup
mariadb_automysqlbackup: yes

# IP address and port to listen on
# bind_address: "127.0.0.1" for local only, "::" for local and remote
mariadb_bind_address: "127.0.0.1"
mariadb_port: 3306

# Maximum connections
mariadb_max_connections: 150

# Replication settings:
# * Enable mariadb_replication_master on master server
# * Enable mariadb_replication_slave on slave server
# * Enable both if for master-master replication setup
mariadb_replication_master: False
mariadb_replication_slave: False

# server ID for replication (needs to be unique in replication setups)
mariadb_server_id: 1

# local replication user (required to be set on masters)
#mariadb_replication_user_local:
#  user: repl
#  host: '%'
#  pass: insecure

# replication master server (required to be set on slaves)
#mariadb_replication_master_server: db.example.org

# remote replication user (required to be set on slaves)
#mariadb_replication_user_remote:
#  user: repl
#  pass: insecure

# MySQLd options
mariadb_mysqld_options: {}

# MySQLd performance options
mariadb_mysqld_performance_options:
  'innodb_buffer_pool_instances': '{{ ansible_processor_vcpus | d(1) }}'
  'innodb_buffer_pool_size':      '{{ (ansible_memtotal_mb / 2) | int }}M'
  'query_cache_type':             '0'
# Prevent double buffering for RAID setups
#'innodb_flush_method':            'O_DIRECT'
# Use RAM buffer for InnoDB write operations
#'innodb_flush_log_at_trx_commit': '0'

# MySQLd character set options
mariadb_mysqld_charset_options:
  'character_set_server': 'utf8'
  'collation_server':     'utf8_general_ci'
  'init_connect':         'SET NAMES utf8'

mariadb_mysqld_network_options:
  'bind_address':    '{{ mariadb_bind_address }}'
  'port':            '{{ mariadb_port }}'
  'max_connections': '{{ mariadb_max_connections }}'

# MySQLd PKI/SSL options (disabled per default)
mariadb_mysqld_pki_options: {}
#  'ssl':
#  'ssl_ca':     '{{ mariadb_ssl_ca if mariadb_ssl_ca|d() else "/etc/letsencrypt/..." }}'
#  'ssl_cert':   '{{ mariadb_ssl_cert if mariadb_ssl_cert|d() else "/etc/letsencrypt/..." }}'
#  'ssl_key':    '{{ mariadb_ssl_key if mariadb_ssl_key|d() else "/etc/letsencrypt/..." }}'
#  'ssl_cipher': '{{ mariadb_ssl_cipher if mariadb_ssl_cihper|d() else "/etc/letsencrypt/..." }}'

mariadb_mysqld_bin_log_options:
  'log_bin':      '/var/log/mysql/mysql-bin.log'
  'log_basename': '{{ ansible_hostname }}'

mariadb_mysqld_repl_options:
  'server_id':    '{{ mariadb_server_id }}'

mariadb_mysqld_options_combined:
  - section: "mysqld"
    options:
      - '{{ mariadb_mysqld_performance_options }}'
      - '{{ mariadb_mysqld_charset_options }}'
      - '{{ mariadb_mysqld_network_options }}'
      - '{{ mariadb_mysqld_pki_options }}'
      - '{{ mariadb_mysqld_bin_log_options if mariadb_replication_master else [] }}'
      - '{{ mariadb_mysqld_repl_options if (mariadb_replication_master or mariadb_replication_slave) else [] }}'
      - '{{ mariadb_mysqld_options }}'

# Munin monitoring
mariadb_munin: False
#mariadb_munin_commands_select_warning: 300
#mariadb_munin_commands_select_critical: 600
```

# License

This Ansible role is licensed under the GNU GPLv3 or later.

# Author

[https://www.systemli.org](https://www.systemli.org)
