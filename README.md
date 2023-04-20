Graylog
==========

The present role :
  - installs graylog server inside a Docker container

The role has been successfully tested on :
  - debian 10 (buster)
  - Ubuntu 22.04.2 LTS

Role variables
--------------

| Variable                                     | Type    | Choices                                                                            | Default                 | Comment         |
|----------------------------------------------|---------|------------------------------------------------------------------------------------|-------------------------|-----------------|
| GRAYLOG_ROOT_PASSWORD_SHA2_password          | string  |                                                                                    | admin                   |                 |
| graylog_port_tcp_syslog                      | string  |                                                                                    | 1514                    |                 |
| graylog_url                                  | string  |                                                                                    | www.graylog.localhost   |                 |
| graylog_server_port                          | string  |                                                                                    | 9000                    |                 |
| graylog_docker_network                       | string  |                                                                                    | graylog                 |                 |
| opensearch_memory                            | string  |                                                                                    | 2g                      |                 |
| opensearch_java_memory_OPTS                  | string  |                                                                                    | '-Xms1g -Xmx1g'         |                 |

Dependencies
------------
  - jq
  - Docker must installed and running for graylog server
  - pwgen

Example Playbook
----------------



Example variables
-----------------



TODO
----


Troubleshoots :
-----------------

If you have this error during starting docker-compose :

- 1 docker-compose version
```
ERROR: The Compose file './docker-compose.yml' is invalid because:
services.graylog.depends_on contains an invalid type, it should be an array
```
just do This
```
apt remove docker-compose -y
python3 -m pip install --upgrade pip
python3 -m pip install docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Mongodb don't start
```
WARNING: MongoDB 5.0+ requires a CPU with AVX support, and your current system does not appear to have that!
  see https://jira.mongodb.org/browse/SERVER-54407
  see also https://www.mongodb.com/community/forums/t/mongodb-5-0-cpu-intel-g4650-compatibility/116610/2
  see also https://github.com/docker-library/mongo/issues/485#issuecomment-891991814
```
This is because your cpu do not support avx

License
-------

GPLv3

Author Information
------------------

Written by Jean-Yves Fournier
