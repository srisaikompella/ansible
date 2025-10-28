Basically ansible creates two groups even if you donot define any groups in your inventory. 
1) all
2) ungrouped

all group contains every host in the inventory file. ungrouped contains only hosts that do not belong to any group.

ex: 

Inventory file:

web2
[webservers]
web1 ansible_host=127.0.0.1 


[servers]
172.16.4.[10:18:2] ansible_user=wavelabs


Command:
ansible ungrouped -i hosts --list-hosts
o/p
 hosts (1):
    web2
since web2 is not part of anygorup it printed web2 as o/p.

```
ansible all -i hosts --list-hosts
```
This will print all the hosts in the inventory.

o/p:
  hosts (7):
    web2
    web1
    172.16.4.10
    172.16.4.12
    172.16.4.14
    172.16.4.16
    172.16.4.18


You can use ranges in the inventory. for example if you check the above inventory under servers we specified range
[10:18:2], meaning from 10 to 18 and width is 2.
so meaning 10, 12, 14, 16, 18

Also when we try to connect to localhost, we can simply specify: localhost ansible_connection=local

instead of specifying web1 ansible_host=127.0.0.1

so if you try to understand "localhost" or "web1" is the alias and the asible_host is thr Ip used to connect to the host.

when you specify web1 ansible_host=127.0.0.1 ansible is using ssh protocol to connect to the rmeote host.

when you specify localhost ansible_connection it uses local connection instead of doing ssh.


Also, say if you have a gorup webservers

[webservers]
ansible_host=10.0.1.2 ansible_user=kasi


You cannot specify the hosts as above. when you use ansible_host in the inventory then you should specify host alias

ex: 
[webservers]
web1 ansible_host=10.0.1.2 ansible_user=kasi

if you dont want to specify host alias then you omit using ansible_host and only specify IP addr as the host
ex:
[webservers]
10.0.1.2 ansible_user=kasi


You can also define the inventory and remote_user settings in the ansible.cfg file.
inventory = ./hosts.yaml
remote_user = kasi

if you create sucj ansible.cfg in your prokect dir or home dir, then you dont have to specify inventory using -i everytime you run ansible commands.

env variable takes more preceedemce for ansible config
export ANSIBLE_CONFIG=~/ansible.cfg


### Variables
3 ways

you can create vars inside a playbook as follows?:
- name: Test custom module
  hosts: all
  vars:
    custom_name: hind
  tasks:
    - name: Print var
      debug:
        msg: "I am  {{ custom_name }}..."


Or else, we can create a vars.yaml

cat vars.yaml

custom_name: hind

then in playbook 
- name: Test custom module
  hosts: all
  var_files:
     - vars.yaml
  tasks:
    - name: Test hind custom module
      hind_module:
        name: maa
    - name: Print var
      debug:
        msg: "I am  {{ custom_name }}..."

3 rd way make it as a task in play

- name: Test custom module
  hosts: all
  tasks:
    - name: include vars
      include_vars:
       - vars.yaml

    - name: Print var
      debug:
        msg: "I am  {{ custom_name }}..."


4th way you can directly specify the vars inside the inventory itself.
- name: Test custom module
  hosts: all
  tasks:
    - name: Test hind custom module
      hind_module:
        name: maa
    - name: Print var
      debug:
        msg: "I am  {{ custom_name }}..."
But in this way you would have to specify this with every host in the inventoryy.


web2
[webservers]
web1 ansible_host=127.0.0.1 custom_name=hind 


We can also specify the vars during the run time

ansible-playbook play.yaml -e "custom_name=hind my_name=maa"


Simple command to use modules on the cli
ansible webservers -m command -a "ls /home/kashi/temp_work/hind-custom-module" 


We can also do
- name: Test custom module
  hosts: {{ nodes }}
  vars:
    custom_name: hind
  tasks:
    - name: Print var
      debug:
        msg: "I am  {{ custom_name }}..."


ansible-playbook play.yml -e "nodes=webservers"

wehre nodes is the variable name and webservers is the inventory name.


## Host vars and group vars
generally we specify host vars and group vars in inventory

[webservers]
web1 ansble_host=10.0.1.2 custom_name=hind
web2 ansible_host=10.0.1.3 custom_name=maa

[db]
node1 
node2
node3

[db:vars]
ansible_user=hind db_name=orc8r

instead of this.
we can create host_vars and group_vars dir

for above ex,
cat host_vars/node1
ansble_host: 10.0.1.2 
custom_name: hind

cat  host_vars/node2
ansble_host: 10.0.1.2
custom_name: maa

cat group_Vars/db
ansible_user: hind 
db_name: orc8r

After we created these dirs, the inventory can be modified as follows:

[webservers]
web1 
web2 

[db]
node1 
node2
node3

as you have noticed the group vars and hostvars in inventory are removed.

NOTE: among group vars and host vars, host vars will takes more preceedence. 

## Targeting specific hosts in the inventory

Suppose in the inventory I have the following hosts:

```
[all]
compute1 ansible_host=172.16.5.174  # ip=10.3.0.1 etcd_member_name=etcd1
compute2 ansible_host=172.16.5.55 # ip=10.3.0.2 etcd_member_name=etcd2
compute3 ansible_host=172.16.5.123  # ip=10.3.0.3 etcd_member_name=etcd3

# ## configure a bastion host if your computes are not directly reachable
# [bastion]
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube_control_plane]
compute1
compute2
compute3

[etcd]
compute1
compute2
compute3


[kube_node:children]
kube_control_plane
[kube_compute]
compute1
compute2
compute3
# compute4
# compute5
# compute6

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_compute
calico_rr
```

So here we have 3 hosts in the all group. SO if we want to target only compute2 and compute3, then we can run the following:

```
ansible-playbook -i invetory.ini playbook.yml --limit "compute2,compute3" 
```
So it will run the playbook against those 2 hosts..

limit says that compute2 and compute3 will be included and all the tasks in your playbook will run on them, But compute1 will be completely skipped. with --limit we can also specify group name .. Say kube_control_plane then all the tasks will be run on that specific group. And we can also specify combinations like this. --limit kube_control_plane:!comoute1 means run on all compute hosts except for compute1.
 