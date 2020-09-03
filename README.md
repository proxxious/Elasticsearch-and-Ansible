# ElasticStack-and-Ansible
Automating ElasticStack deployment with Ansible

**ANSIBLE INSTALLATION**

Follow the below guide to setup Ansible
https://computingforgeeks.com/setup-elasticsearch-cluster-on-centos-ubuntu-with-ansible/

**NOTE**
 Copy the SSH ID from your control node to the remote host
 
 `ssh-copy-id username@remote_host`

**Create your playbook**

elk.yml

```
hosts: elk1
  roles:
    - role: elastic.elasticsearch
  vars:
    es_enable_xpack: true
    es_data_dirs:
      - "/data/elasticsearch/data"
    es_log_dir: "/data/elasticsearch/logs"
    es_heap_size: "1g"
    es_config:
      cluster.name: "elk-cluster"
      cluster.initial_master_nodes: "192.168.17.167:9300,192.168.17.170:9300"
      discovery.seed_hosts: "192.168.17.167:9300,192.168.17.170:9300"
      http.port: 9200
      node.data: true
      node.master: true
      bootstrap.memory_lock: false
      network.host: '0.0.0.0'
    es_plugins:
     - plugin: ingest-attachment
    es_api_basic_auth_username: elastic
    es_api_basic_auth_password: changeme
    es_enable_http_ssl: true
    es_enable_transport_ssl: true
    es_ssl_keystore: "/root/http.p12"
    es_ssl_truststore: "/root/http.p12"
    es_ssl_keystore_password: "qwerty"
    es_ssl_truststore_password: "qwerty"
    es_ssl_verification_mode: certificate

- hosts: elk2
  roles:
    - role: elastic.elasticsearch
  vars:
    es_enable_xpack: true
    es_data_dirs:
      - "/data/elasticsearch/data"
    es_log_dir: "/data/elasticsearch/logs"
    es_heap_size: "1g"
    es_config:
      cluster.name: "elk-cluster"
      cluster.initial_master_nodes: "192.168.17.167:9300,192.168.17.170:9300"
      discovery.seed_hosts: "192.168.17.167:9300,192.168.17.170:9300"
      http.port: 9200
      node.data: true
      node.master: true
      bootstrap.memory_lock: false
      network.host: '0.0.0.0'
    es_plugins:
     - plugin: ingest-attachment
    es_api_basic_auth_username: elastic
    es_api_basic_auth_password: changeme
    es_enable_http_ssl: true
    es_enable_transport_ssl: true
    es_ssl_keystore: "/root/http.p12"
    es_ssl_truststore: "/root/http.p12"
    es_ssl_keystore_password: "qwerty"
```
Create your hosts file

hosts

```
[elk-nodes]
elk1
elk2
```
**Run the Ansible playbook**

ansible-playbook -i hosts elk.yml
