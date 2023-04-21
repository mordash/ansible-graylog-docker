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
| graylog_email_enabled                     | bool    | true/false                                                                                   | false                   |                 |
| graylog_email_hostname | string | | smtp.gmail.com | |
| graylog_email_port | int | | 587 | |
| graylog_email_use_auth | bool | true/false | true | |
| graylog_email_use_tls | bool | true/false | true | |
| graylog_email_auth_username | string | | my_mail@gmail.com | |
| graylog_email_auth_password | string | | my_password | |
| graylog_email_from_email | string | | my_mail@gmail.com | |
| graylog_email_subject_prefix | string | | [graylog] | |
| opensearch_memory                            | string  |                                                                                    | 2g                      |                 |
| opensearch_java_memory_OPTS                  | string  |                                                                                    | '-Xms1g -Xmx1g'         |                 |

Dependencies
------------
  - jq
  - Docker must installed and running for graylog server
  - pwgen

Must do :
------------
Create a lvm partition for docker : /var/lib/docker

Example Playbook
----------------
```yml
---
- hosts: localhost
  gather_facts: yes
  ignore_errors: "{{ ansible_check_mode }}" # ignore errors only in check mode !

  roles:
    - { role: ansible-graylog-docker,          tags: ['ansible-graylog-docker'] }
```

Exemple graylog-proxy-pass nginx on another docker server :
------------
```
events {

}

http {
  server {
    listen 80;

    location / {
      proxy_pass http://<ip_graylog_server>:9000;
    }
  }
}
```
```
version: '3.7'

networks:
  traefik:
    external: true

services:
  graylog-proxy-pass:
    image: nginx:latest
    container_name: graylog-proxy-pass
    restart: unless-stopped
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - 127.0.0.1:82:80
    networks:
      - traefik
    labels:
      traefik.docker.network: traefik
      traefik.enable: true
      traefik.http.routers.graylog.entrypoints: https
      traefik.http.routers.graylog.rule: Host(`<graylog_url>`)
      traefik.http.routers.graylog.tls: true
      traefik.http.routers.graylog.tls.certresolver: letsencrypt
      traefik.frontend.auth.basic.users: "{{ _auth_users|join(',') }}"
      traefik.http.services.graylog.loadbalancer.server.port: 80
```

Example variables
-----------------


Requirements
-----------------

redirect rsyslog apache like this :
```
:syslogtag, startswith, "vhost_apache" @@graylog_server_ip:graylog_port_tcp_syslog;RSYSLOG_LongTagForwardForma
```

TODO
----

  - documentation
    - review / enhance documentation
  - content pack
    - add options to import content pack
  - log format
    - add options to change logformat haproxy/apache2

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
