# [ansible](https://www.ansible.com/)

## installation
```
yum install ansible
apt install ansible
```
```
pip install ansible 
```
remote machine should have 'python' - 'gather_facts: False' otherwise

## [examples](https://github.com/ansible/ansible-examples)
## [examples 2](https://github.com/mmumshad/ansible-training-answer-keys)
## [examples 3](https://github.com/mmumshad/ansible-training-answer-keys-2)

## ansible configuration places
* path variable $Ansible_Config
* ./ansible.cfg
* ~/ansible.cfg
* /etc/ansible/ansible.cfg

## execute ansible-playbook with external paramters, bash script ansible-playbook with parameters
```
ansible-playbook -i inventory.ini playbook.yml --extra-vars "$*"
```
with path to file for external parameters
```
ansible-playbook -i inventory.ini playbook.yml --extra-vars @/path/to/var.properties
```

## check is it working, ad-hoc command
```
ansible remote* -i inventory.ini -m "ping"
ansible remote* -i inventory.ini --module-name "ping"
```
```
ansible remote* -i inventory.ini -a "hostname"
```

## loop example
```
    - name: scripts {{ item }}
      template:
        mode: 0777 
        src: "templates/{{ item }}" 
        dest: "{{ root_folder }}/{{ item }}" 
      loop:
        - "start-all.sh"
        - "status.sh"
        - "stop-all.sh"
```

## repeat execution
```
--limit {playbookfile}.retry
```

## start with task, execute from task, begin with task, skip previous tasks
```
ansible-playbook playbook.yml --start-at-task="name of the start to be started from"
```

## replace variables inside file to dedicated file, move vars to separate file
* before
```
   vars:
      db_user: my_user
      db_password: my_password
      ansible_ssh_pass: my_ssh_password 
      ansible_host: 192.168.1.14
```
* after 
*( 'vars' block is empty )*
filepath: 
```
./host_vars/id_of_the_server
```
or groupvars:
```
./group_vars/id_of_the_group_into_square_brakets
```
code
```
db_user: my_user
db_password: my_password
ansible_ssh_pass: my_ssh_password 
ansible_host: 192.168.1.14
```

## move code to separate file, tasks into file
cut code from original file and paste it into separate file ( with appropriate alignment !!! ),
write instead of the code:
```
    - include: path_to_folder/path_to_file
```
approprate file should be created:
```
./path_to_folder/path_to_file
```
## debug module
```
  - debug:
      msg: "print variable: {{  my_own_var }}"
```

## conditions "when"
TBD

# error handling, try catch
---
## stop execution of steps (of playbook) when at least one server will throw error
```
  any_errors_fatal:true
```

## not to throw error for one certain task
```
 - mail:
     to: 1@yahoo.com
     subject: info
     body: das ist information
   ignore_errors: yes
```

## fail when, fail by condition, parse log file for errors
```
  - command: cat /var/log/server.log
    register: server_log_file
    failed_when : "'ERROR' in server_log_file.stdout"
```

# template, Jinja2 templating
default value
```
default path is {{ my_custom_path | default("/opt/program/script.sh") }}
```
operation with list
```
{{ [1,2,3] | min }}
{{ [1,2,3] | max }}
{{ [1,2,3] | first }}
{{ [1,2,3] | last }}
{{ [1,2,3,2,3,] | unique }}
{{ [1,2,3] | union([1,2]) }}
{{ [1,2,3] | intersect([3]) }}
{{ 100 | random }}
{{ ["space", "separated", "value"] | join(" ") }}
```
file name from path (return 'script.sh')
```
{{ "/etc/program/script.sh" | basename }}
```

# [plugins](https://github.com/ansible/ansible/tree/devel/lib/ansible/plugins/)
example of plugin
```
{{ list_of_values | average }}
```
python code for plugin
```
dev average(list):
    return sum(list) / float(len(list))
    
class AverageModule(object):
    def filters(self):
        return {'average': average}
```
execution
```
export ANSIBLE_FILTER_PLUGINS=/full/path/to/folder/with/plugin
ansible-playbook playbook.yml
```

## lookup
replace value from file with special format
```
{{ lookup('csvfile', 'web_server file=credentials.csv delimiter=,') }}
{{ lookup('ini', 'password section=web_server file=credentials.ini') }}
```

# inventory file
---
## inventory file, inventory file with variables, [rules](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
```
[remote_ssh]
172.28.128.3     ansible_connection=ssh   ansible_port=22   ansible_user=tc     ansible_password=tc
```
## dynamic inventory
python inventory.py (with 'py' extension) instead of txt
```
import json
data = {"databases": {"hosts": ["host1", "host2"], "vars": {"ansible_ssh_host":"192.168.10.12", "ansible_ssh_pass":"Passw0rd"} }}
print(json.dumps(data))
```
also next logic should be present
```
inventory.py --list
inventory.py --host databases
```
[prepared scripts](https://github.com/ansible/ansible/tree/devel/contrib/inventory)

## inventory file with variables ( python Jinja templating)
```
[remote_ssh]
172.28.128.3     ansible_connection=ssh   ansible_port=22   ansible_user=tc     ansible_password=tc   http_port=8090
```
playbook usage:
```
'{{http_port}}'
```

## execution with inventory examples
for one specific host without inventory file 
```
ansible-playbook playbook.yml -i 10.10.10.10
```
with inventory file
```
ansible-playbook -i inventory.ini playbook.yml 
```

# strategy
---
```
  strategy: linear
```
* linear ( default )
*after each step waiting for all servers*
* free
*independently for all servers - someone can finish installation significantly earlier than others*

additional parameter - specify amount of servers to be executed at the time ( for default strategy only )
```
  serial: 3
```
```
  serial: 20%
```
```
  serial: [5,15,20]
```

default value "serial" into configuration **ansible.cfg**
```
forks = 5
```

# async execution, nowait task, command execution
**not all modules support this operation**
execute command in asynchronous mode ( with preliminary estimation 120 sec ), 
with default poll result of the command - 10 ( seconds )
```
  async: 120
```
execute command in asynchronous mode ( with preliminary estimation 120 sec ), 
with poll result of the command - 60 ( seconds )
```
  async: 120
  poll: 60
```
execute command and forget, not to wait for execution
```
  async: 120
  poll: 0
```
execute command in asynchronous mode, 
register result
checking result at the end of the file
```
- command: /opt/my_personal_long_run_command.sh
  async: 120
  poll: 0
  register: custom_command_result
  
- name: check status result
  async_status: jid={{ custom_command_result.ansible_job_id }}
  register: command_result
  until: command_result.finished
  retries: 20
```


# roles
---
## init project ansible-galaxy, create new role, init role
execute code into your project folder './roles'
```
ansible-galaxy init {project/role name}
```
result:
```
./roles/{project/role name}
    /defaults
    /handlers
    /meta
    /tasks
    /tests
    /vars
```
insert into code
```
  roles:
  - {project/role name}
```
all folders of the created project will be applied to your project ( tasks, vars, defaults )
*in case of manual creation - only necessary folders can be created*

## ansible search for existing role
```
ansible-galaxy search {project/role name}
```

## import existing roles from [ansible galaxy](https://galaxy.ansible.com/list)
```
cd roles
ansible-galaxy import {name of the project/role}
```
insert into code
```
  roles:
  - {project/role name}
```
all folders of the imported project will be applied to your project ( tasks, vars, defaults )

## export 
create/update file:
```
./roles/{project/role name}/meta/main.yml
```

## console output with applied roles should looks like
```
TASK [{project/role name}: {task name}] ***********************************
```
for example
```
TASK [java : install java with jdbc libraries] ***********************************
```
# file encryption, vault
```
ansible-vault encrypt inventory.txt
ansible-vault view inventory.txt
ansible-vault create inventory.txt
```
ask password via command line
```
ansible-playbook playbook.yml -i inventory.txt -ask-vault-pass
```
file should contain the password
```
ansible-playbook playbook.yml -i inventory.txt -vault-password-file ./file_with_pass.txt
```
script should return password
```
ansible-playbook playbook.yml -i inventory.txt -vault-password-file ./file_with_pass.py
```

# modules
[list of all modules](https://docs.ansible.com/ansible/devel/modules/list_of_all_modules.html)
[custom module playground](https://ansible-playable.com)
[custom module creation doc](docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html)

### [apt](https://docs.ansible.com/ansible/latest/modules/apt_module.html), python installation 
```
- name: example of apt install 
  apt: name='{{ item }}' state=installed
  with_items:
    - python
    - python-setuptools
    - python-dev
    - build-essential
    - python-pip
```
### [service](https://docs.ansible.com/ansible/latest/modules/service_module.html)
```
- name: example of start unix service
  service:
    name: mysql
    state: started
    enabled: yes
```

### [pip](https://docs.ansible.com/ansible/latest/modules/pip_module.html)
```
- name: manage python packages via pip 
  pip:
    name: flask
```

### echo
```
- debug:
    msg: ">>> {{ data_portal_deploy_folder }}/data-portal.jar"
```

### TBD
* system
* commands
* database
* cloud
* windows


# [ansible awx](https://github.com/ansible/awx)

# issues

## fingerprint checking
```
fatal: [172.28.128.4]: FAILED! => {"msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this.  Please add this host's fingerprint to your known_hosts file to manage this host."}
```
resolution
```
export ANSIBLE_HOST_KEY_CHECKING=False
ansible-playbook -i inventory.ini playbook-directory.yml
```
