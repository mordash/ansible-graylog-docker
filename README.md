Graylog
==========

The present role :
  - installs graylog server inside a Docker container

The role has been successfully tested on :
  - debian 10 (buster)

Role variables
--------------

| Variable                                     | Type    | Choices                                                                            | Default                 | Comment         |
|----------------------------------------------|---------|------------------------------------------------------------------------------------|-------------------------|-----------------|
| GRAYLOG_ROOT_PASSWORD_SHA2_password             | string  |                                                                                    | admin               |                 |
| graylog_port_tcp_syslog             | string  |                                                                                    | 1514               |                 |

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


Troubles :
-----------------
If you have this error :
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


License
-------

GPLv3

Author Information
------------------

Written by Jean-Yves Fournier
