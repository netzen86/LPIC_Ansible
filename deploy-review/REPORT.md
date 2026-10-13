## LAB02

файл конфигурации
```
[defaults]
inventory = ~/tasks/deploy-review/inventory
remote_user = devops
ask_pass = false

[privilege_escalation]
become = false
become_method = sudo
become_user = root
become_ask_pass = false
```

выполнение adhoc команды `ansible all -m command -a id` с параметром `become = false` в конфиге
```
servera.lab.example.com | CHANGED | rc=0 >>
uid=1001(devops) gid=1001(devops) группы=1001(devops) контекст=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
serverd.lab.example.com | CHANGED | rc=0 >>
uid=1001(devops) gid=1001(devops) группы=1001(devops) контекст=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
serverb.lab.example.com | CHANGED | rc=0 >>
uid=1001(devops) gid=1001(devops) группы=1001(devops) контекст=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
serverc.lab.example.com | CHANGED | rc=0 >>
uid=1001(devops) gid=1001(devops) группы=1001(devops) контекст=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

выполнение adhoc команды `ansible all -m command -a id` с параметром `become = true` в конфиге, видим что при этом команда выполнилась от пользователя `root`
```
serverb.lab.example.com | CHANGED | rc=0 >>
uid=0(root) gid=0(root) группы=0(root) контекст=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
serverd.lab.example.com | CHANGED | rc=0 >>
uid=0(root) gid=0(root) группы=0(root) контекст=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
servera.lab.example.com | CHANGED | rc=0 >>
uid=0(root) gid=0(root) группы=0(root) контекст=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
serverc.lab.example.com | CHANGED | rc=0 >>
uid=0(root) gid=0(root) группы=0(root) контекс
```

`ansible all -b -m copy -a "content='This server is managed by Ansible. \n Second try' dest=/etc/motd"`

выполнение команды без ключа -b
```
serverd.lab.example.com | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "checksum": "dbef34529341290d82d21c0680219f77cb33d5a2",
    "msg": "Destination /etc not writable"
}
```

выполнение команды с ключем -b
```
serverc.lab.example.com | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "checksum": "dbef34529341290d82d21c0680219f77cb33d5a2",
    "dest": "/etc/motd",
    "gid": 0,
    "group": "root",
    "md5sum": "3f33c97e84748b1f97fee531e6e0ea41",
    "mode": "0644",
    "owner": "root",
    "secontext": "system_u:object_r:etc_t:s0",
    "size": 47,
    "src": "/home/devops/.ansible/tmp/ansible-tmp-1791829102.4657242-9218-72382359528861/source",
    "state": "file",
    "uid": 0
}

servera.lab.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "checksum": "dbef34529341290d82d21c0680219f77cb33d5a2",
    "dest": "/etc/motd",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/etc/motd",
    "secontext": "system_u:object_r:etc_t:s0",
    "size": 47,
    "state": "file",
    "uid": 0
}
```

`ansible all -m command -a "cat /etc/motd"`

```
servera.lab.example.com | CHANGED | rc=0 >>
This server is managed by Ansible. 
 Second try
serverd.lab.example.com | CHANGED | rc=0 >>
This server is managed by Ansible. 
 Second try
serverb.lab.example.com | CHANGED | rc=0 >>
This server is managed by Ansible. 
 Second try
serverc.lab.example.com | CHANGED | rc=0 >>
This server is managed by Ansible. 
 Second try
```

## LAB03

на сервере `serverb.lab.example.com` не резолвился адрес `http://materials.example.com/labs/playbook-review/index.php` прописал в hosts

скрипт не мог приконнектиться к базе без пароля, 

добавил в базу нового пользователя, 
```
mysql -u root

CREATE USER 'webuser'@'localhost' IDENTIFIED BY 'password123';
GRANT ALL PRIVILEGES ON *.* TO 'webuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

```

исправил скрипт

```
<?php
$servername = "localhost";
$username = "webuser";
$password = "password123";

// Create connection
$conn = mysqli_connect($servername, $username, $password);

// Check connection
if (!$conn) {
  http_response_code(500);
} else {
  http_response_code(200);
}
```

```
PLAY [Enable internet services] ****************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [latest firewalld version installed] ******************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [latest httpd version installed] **********************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [latest mariadb-server version installed] *************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [latest php version installed] ************************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [latest php-mysqlnd version installed] ****************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [firewalld is enable and ruuning] *********************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [Ensure HTTP traffic is allowed] **********************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [httpd is enable and ruuning] *************************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [mariadb-server is enable and started] ****************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

TASK [Download index.php] **********************************************************************************************************************************************************************************************
ok: [serverb.lab.example.com]

PLAY [Check web server] ************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [test web service on serverb] *************************************************************************************************************************************************************************************
ok: [localhost]

PLAY RECAP *************************************************************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
serverb.lab.example.com    : ok=11   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```