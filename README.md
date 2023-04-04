Graylog
==========

The present role :
  - installs graylog server inside a Docker container

The role has been successfully tested on :


Role variables
--------------

| Variable                                     | Type    | Choices                                                                            | Default                 | Comment         |
|----------------------------------------------|---------|------------------------------------------------------------------------------------|-------------------------|-----------------|
| GRAYLOG_ROOT_PASSWORD_SHA2_password             | string  |                                                                                    | admin               |                 |


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
