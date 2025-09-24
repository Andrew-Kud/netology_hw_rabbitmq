# Домашнее задание к занятию "`Очереди RabbitMQ`" - `Кудряшов Андрей`


### Задание 1

<img width="1276" height="1435" alt="1" src="https://github.com/user-attachments/assets/dc399b5d-d4f1-4c15-9a6d-abba56e11360" />


---

### Задание 2

<img width="1278" height="1400" alt="2 1" src="https://github.com/user-attachments/assets/e730c75a-4bf3-4273-af96-b4ddbbe7b225" />

<img width="1285" height="1398" alt="2 2" src="https://github.com/user-attachments/assets/07e29613-ad8d-4186-bc02-bf515902627f" />

<img width="1281" height="1392" alt="2 3" src="https://github.com/user-attachments/assets/4655cfc0-c98f-44c6-af53-9ee7f3e8b558" />

<img width="1273" height="486" alt="2 4" src="https://github.com/user-attachments/assets/cbfe1b58-7456-4e56-9e4c-4ee1cb4ea21b" />


---

### Задание 3

<img width="2558" height="1439" alt="3 1" src="https://github.com/user-attachments/assets/deea27ff-ca15-4628-9abe-8ae32e060b17" />

<img width="2556" height="1438" alt="3 2" src="https://github.com/user-attachments/assets/1348bb86-3ae7-4e58-8755-0f774cff14e0" />

<img width="2557" height="1439" alt="3 3" src="https://github.com/user-attachments/assets/a1fff0d3-1b0d-4bc4-9645-e9fe21bf22e9" />

<img width="2560" height="468" alt="3 4" src="https://github.com/user-attachments/assets/5bf6fe84-d765-4f86-b76b-ac9b792b3664" />

<img width="2559" height="1438" alt="3 5" src="https://github.com/user-attachments/assets/c91c4c15-3fa1-4cf5-864e-3ed0a4ef02a7" />

<img width="480" height="104" alt="4 1" src="https://github.com/user-attachments/assets/041a5a0d-da65-4947-9246-ad021b4c40a3" />

<img width="1263" height="1347" alt="4 2" src="https://github.com/user-attachments/assets/1f463c98-f22d-4142-875e-f41949f73f52" />

<img width="1279" height="1353" alt="4 3" src="https://github.com/user-attachments/assets/625b0b81-19cf-4f09-80bc-708bd7da8b28" />


---

### Задание 4

<img width="2560" height="1440" alt="5 1" src="https://github.com/user-attachments/assets/d6f4e121-e716-4690-87de-794c990ee1ca" />

<img width="2559" height="1440" alt="5" src="https://github.com/user-attachments/assets/ae091b4f-e77d-4819-81d4-ad5eb51b33e3" />


Структура:

rabbitmq-ha-ansible/
├── ansible.cfg
├── inventory.ini
├── playbook.yml
├── group_vars/
│   └── rmq_cluster.yml
└── roles/
    └── rabbitmq/
        ├── tasks/
        │   └── main.yml
        └── handlers/
            └── main.yml

ansible.cfg
```
[defaults]
inventory = inventory.ini
remote_user = kudryashov
host_key_checking = False
roles_path = ./roles
```

invertory.ini
```
[rmq_nodes]
netology2 ansible_host=192.168.10.215
netology3 ansible_host=192.168.10.216
netology4 ansible_host=192.168.10.217

[rmq_cluster:children]
rmq_nodes

[rmq_cluster:vars]
ansible_user=kudryashov
ansible_ssh_private_key_file=/home/kudryashov/.ssh/id_rsa
```

playbook.yml
```
---
- name: Install and Configure RabbitMQ Cluster
  hosts: rmq_cluster
  become: yes
  roles:
    - rabbitmq
```

/group_vars/rmq_cluster.yml
```
---
rabbitmq_user: test_admin
rabbitmq_password: test_admin
rabbitmq_erlang_cookie: "4H+ISECo1TJUXsgfdty7s75B3G++1fMY"
rabbitmq_version: "4.1.4"
```

/roles/rabbitmq/tasks/main.yml
```
---
- name: Add RabbitMQ signing key
  become: yes
  apt_key:
    url: https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc
    state: present

- name: Install prerequisites for RabbitMQ
  become: yes
  apt:
    name:
      - curl
      - gnupg
      - apt-transport-https
    state: present

- name: Download and install RabbitMQ team GPG key
  become: yes
  shell: |
    curl -1sLf 'https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA' | gpg --dearmor | tee /usr/share/keyrings/com.rabbitmq.team.gpg > /dev/null
  args:
    creates: /usr/share/keyrings/com.rabbitmq.team.gpg

- name: Add RabbitMQ and Erlang repositories
  become: yes
  copy:
    dest: /etc/apt/sources.list.d/rabbitmq.list
    content: |
      deb [arch=amd64 signed-by=/usr/share/keyrings/com.rabbitmq.team.gpg] https://deb1.rabbitmq.com/rabbitmq-erlang/ubuntu/{{ ansible_distribution_release }} {{ ansible_distribution_release }} main
      deb [arch=amd64 signed-by=/usr/share/keyrings/com.rabbitmq.team.gpg] https://deb2.rabbitmq.com/rabbitmq-erlang/ubuntu/{{ ansible_distribution_release }} {{ ansible_distribution_release }} main

      deb [arch=amd64 signed-by=/usr/share/keyrings/com.rabbitmq.team.gpg] https://deb1.rabbitmq.com/rabbitmq-server/ubuntu/{{ ansible_distribution_release }} {{ ansible_distribution_release }} main
      deb [arch=amd64 signed-by=/usr/share/keyrings/com.rabbitmq.team.gpg] https://deb2.rabbitmq.com/rabbitmq-server/ubuntu/{{ ansible_distribution_release }} {{ ansible_distribution_release }} main
    owner: root
    group: root
    mode: '0644'

- name: Update apt cache
  become: yes
  apt:
    update_cache: yes

- name: Install Erlang packages
  become: yes
  apt:
    name:
      - erlang-base
      - erlang-asn1
      - erlang-crypto
      - erlang-eldap
      - erlang-ftp
      - erlang-inets
      - erlang-mnesia
      - erlang-os-mon
      - erlang-parsetools
      - erlang-public-key
      - erlang-runtime-tools
      - erlang-snmp
      - erlang-ssl
      - erlang-syntax-tools
      - erlang-tftp
      - erlang-tools
      - erlang-xmerl
    state: present

- name: Install RabbitMQ server
  become: yes
  apt:
    name: rabbitmq-server
    state: present

- name: Start and enable RabbitMQ
  become: yes
  systemd:
    name: rabbitmq-server
    state: started
    enabled: yes

- name: Enable management plugin
  become: yes
  command: rabbitmq-plugins enable rabbitmq_management
  args:
    creates: /etc/rabbitmq/enabled_plugins

- name: Check if admin user exists
  become: yes
  command: rabbitmqctl list_users --silent
  register: users_list
  changed_when: false

- name: Create admin user
  become: yes
  command: rabbitmqctl add_user {{ rabbitmq_user }} {{ rabbitmq_password }}
  when: "rabbitmq_user not in users_list.stdout"

- name: Set admin tags
  become: yes
  command: rabbitmqctl set_user_tags {{ rabbitmq_user }} administrator
  args:
    creates: /tmp/rmq_tags_set_{{ rabbitmq_user }}

- name: Set permissions
  become: yes
  command: rabbitmqctl set_permissions -p / {{ rabbitmq_user }} ".*" ".*" ".*"
  args:
    creates: /tmp/rmq_permissions_set_{{ rabbitmq_user }}

- name: Configure /etc/hosts for cluster nodes (add all cluster nodes)
  become: yes
  lineinfile:
    path: /etc/hosts
    line: "{{ hostvars[item].ansible_host }} {{ item }}"
    state: present
  loop: "{{ groups['rmq_nodes'] }}"
  notify: Restart RabbitMQ

- name: Ensure node can resolve its own hostname (add self to 127.0.0.1)
  become: yes
  lineinfile:
    path: /etc/hosts
    line: "127.0.0.1 {{ inventory_hostname }}"
    state: present
    insertafter: '^127\.0\.0\.1\s+localhost'
  notify: Restart RabbitMQ

- name: Set Erlang cookie
  become: yes
  copy:
    content: "{{ rabbitmq_erlang_cookie }}"
    dest: /var/lib/rabbitmq/.erlang.cookie
    owner: rabbitmq
    group: rabbitmq
    mode: '600'
  notify: Restart RabbitMQ

- name: Wait for RabbitMQ to restart after hosts update
  become: yes
  wait_for:
    port: 5672
    host: "{{ ansible_host }}"
    delay: 5
    timeout: 30

- name: Wait for RabbitMQ to be ready
  become: yes
  command: rabbitmqctl status
  register: rmq_status
  until: rmq_status.rc == 0
  retries: 20
  delay: 10

- name: Stop app on non-first nodes
  become: yes
  command: rabbitmqctl stop_app
  when: inventory_hostname != groups['rmq_nodes'][0]

- name: Reset node on non-first nodes
  become: yes
  command: rabbitmqctl reset
  when: inventory_hostname != groups['rmq_nodes'][0]

- name: Join cluster
  become: yes
  command: "rabbitmqctl join_cluster rabbit@{{ groups['rmq_nodes'][0] }}"
  when: inventory_hostname != groups['rmq_nodes'][0]

- name: Start app on non-first nodes
  become: yes
  command: rabbitmqctl start_app
  when: inventory_hostname != groups['rmq_nodes'][0]

- name: Enable required feature flags for RabbitMQ 4.1
  become: yes
  command: rabbitmqctl enable_feature_flag {{ item }}
  loop:
    - quorum_queue
    - stream_queue
    - implicit_default_bindings
    - virtual_host_metadata
    - maintenance_mode_status
    - message_containers
    - classic_mirrored_queue_version
  ignore_errors: yes
```

/roles/rabbitmq/handlers/main.yml
```
---
- name: Restart RabbitMQ
  become: yes
  systemd:
    name: rabbitmq-server
    state: restarted
```

---

