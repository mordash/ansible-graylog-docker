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



License
-------

GPLv3

Author Information
------------------

Written by Jean-Yves Fournier
