Graylog
==========

The present role :
  - installs graylog server inside a Docker container

The role has been successfully tested on :
  - debian 10 (buster)
  - debian 11 (bullseye)
  - Ubuntu 22.04.2 LTS

Role variables
--------------

| Variable                                     | Type    | Choices                                                                            | Default                 | Comment         |
|----------------------------------------------|---------|------------------------------------------------------------------------------------|-------------------------|-----------------|
| graylog_root_password_sha2_password          | string  |                                                                                    | admin                   |                 |
| graylog_port_tcp_syslog                      | string  |                                                                                    | 1514                    |                 |
| graylog_url                                  | string  |                                                                                    | www.graylog.localhost   |                 |
| graylog_server_port                          | string  |                                                                                    | 9000                    |                 |
| graylog_docker_network                       | string  |                                                                                    | graylog                 |                 |
| graylog_port_tcp_syslog_custom               | list    |                                                                                    | undefined               |                 |
| graylog_port_ucp_syslog_custom               | list    |                                                                                    | undefined               |                 |
| graylog_email_enabled                        | bool    | true/false                                                                         | false                   |                 |
| graylog_email_hostname                       | string  |                                                                                    | smtp.gmail.com          |                 |
| graylog_email_port                           | int     |                                                                                    | 587                     |                 |
| graylog_email_use_auth                       | bool    | true/false                                                                         | true                    |                 |
| graylog_email_use_tls                        | bool    | true/false                                                                         | true                    |                 |
| graylog_email_auth_username                  | string  |                                                                                    | my_mail@gmail.com       |                 |
| graylog_email_auth_password                  | string  |                                                                                    | my_password             |                 |
| graylog_email_from_email                     | string  |                                                                                    | my_mail@gmail.com       |                 |
| graylog_email_subject_prefix                 | string  |                                                                                    | [graylog]               |                 |
| graylog_http_external_uri_var                | string  |                                                                                    | {{ graylog_url }}       |                 |
| opensearch_memory                            | string  |                                                                                    | 2g                      |                 |
| opensearch_java_memory_opts                  | string  |                                                                                    | '-Xms1g -Xmx1g'         |                 |
| geoip_accountid                              | int     |                                                                                    | AccountID               |                 |
| geoip_licensekey                             | string  |                                                                                    | YOUR_LICENSE_KEY_HERE   |                 |
| geoip_editionids                             | string  |                                                                                    | GeoLite2-ASN GeoLite2-City |              |
| geoip_arch                                   | string  |                                                                                    | amd64                   |                 |
| geoip_database_directory                     | string  |                                                                                    | /var/lib/docker/volumes/graylog_data/_data/geoip  |                 |
| docker_image_mongodb                         | string  | latest/'5.0'                                                                       | '5.0'                   |                 |
| docker_image_opensearch                      | string  | latest/'2.4.0'                                                                     | '2.4.0'                 |                 |
| docker_image_graylog                         | string  | latest/'5.0'                                                                       | '5.0'                   |                 |


Dependencies
------------
  - jq
  - Docker must installed and running for graylog server
  - pwgen

Must do :
------------
- Create a lvm partition for docker : /var/lib/docker
- Account GEOIP 'see your preset here https://www.maxmind.com/en/accounts/current/license-key/GeoIP.conf'

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
      traefik.http.routers.graylog.entrypoints: websecure
      traefik.http.routers.graylog.rule: Host(`<graylog_url>`)
      traefik.http.routers.graylog.tls: true
      traefik.http.routers.graylog.tls.certresolver: letsencrypt
      traefik.frontend.auth.basic.users: "{{ _auth_users|join(',') }}"
      traefik.http.services.graylog.loadbalancer.server.port: 80
```

Example variables
-----------------
```
---
graylog_port_tcp_syslog_custom:
  - 10000
  - 10010
```


Requirements
-----------------
***apache***

redirect rsyslog apache like this :
```
:syslogtag, startswith, "vhost_apache" @@graylog_server_ip:graylog_port_tcp_syslog;RSYSLOG_LongTagForwardForma
```

***haproxy***

redirect rsyslog haproxy :
```
if ( ( $syslogfacility-text == "local0" ) and ( $syslogseverity-text == "info" ) and ( $syslogtag startswith "haproxy") ) then @@graylog_server_ip:graylog_port_tcp_syslog;RSYSLOG_LongTagForwardFormat```
```
logformat haproxy :
```
log-format %ci:%cp\ -\ [%Tl]\ %{+Q}r\ %ST\ %B\ [%{+Q}hrl]\ ---\ %{+Q}[var(txn.SCHEME)]\ %{+Q}HV\ [%b/%s]\ [%{+Q}hsl]\ %TR/%Tw/%Tc/%Tr/%Ta
```
grok pattern:
- HAPROXY_LOG
```
%{DATA:source}%{SPACE}%{SYSLOGPROG}:%{SPACE}%{IP:client_ip}:%{INT:client_port}%{GREEDYDATA}\[%{HAPROXYDATE:accept_date}\]%{SPACE}%{HAPROXY_REQUEST}%{SPACE}%{INT:http_status_code}%{SPACE}%{NOTSPACE:bytes_read}%{SPACE}\["%{DATA:X-Forwarded-For}"%{SPACE}"%{DATA:CF-Connecting-IP}"%{SPACE}"%{DATA:request_header_referer}"%{SPACE}"%{DATA:request_header_user_agent}"%{SPACE}"%{HOSTNAME}"\]%{GREEDYDATA}"%{DATA:http_proto}"%{SPACE}"HTTP/%{NUMBER:http_version}"%{SPACE}\[%{NOTSPACE:backend_name}/%{NOTSPACE:server_name}\]%{SPACE}\[%{DATA:captured_response_headers}\]%{SPACE}%{INT:time_1_client_request_ms}/%{INT:time_2_srv_queue_ms}/%{INT:time_3_tcp_backend_connect_ms}/%{INT:time_4_response_ms}/%{INT:time_5_total_session_duration_ms}
```
- HAPROXY_REQUEST
```
"%{WORD:http_verb}%{DATA:rawrequest}HTTP/%{NUMBER:http_version}"
```
- HAPROXYDATE
```
%{MONTHDAY:haproxy_monthday}/%{MONTH:haproxy_month}/%{YEAR:haproxy_year}:%{HAPROXYTIME:haproxy_time}.%{INT:haproxy_milliseconds}
```
- HAPROXYTIME
```
(?!<[0-9])%{HOUR:haproxy_hour}:%{MINUTE:haproxy_minute}(?::%{SECOND:haproxy_second})(?![0-9])
```

- Apache2 error_log ???
```
        Regex  ^\[[^ ]* (?<time>[^\]]*)\] \[(?<level>[^\]]*)\](?: \[pid (?<pid>[^\]]*)\])?( \[client (?<client>[^\]]*)\])? (?<message>.*)$
```
https://www.xtivia.com/blog/k8s-loggings-graylog-fluent-bit/


TODO
----

  - documentation
    - review / enhance documentation
  - content pack
    - add options to import content pack
  - log format
    - add options to change logformat haproxy/apache2
  - tester https://github.com/fluent/fluent-bit
  - tester log errors apache2 ou haproxy

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
