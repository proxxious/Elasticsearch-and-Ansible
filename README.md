# Elasticsearch-and-Ansible
Automating Elasticsearch deployment with Ansible
     
     This is the guide is a two(2) ElasticStack node with one(1) control node for Ansible
     
Prerequisite
     
     Install java

**ANSIBLE INSTALLATION**

Follow the below guide to setup Ansible
https://computingforgeeks.com/setup-elasticsearch-cluster-on-centos-ubuntu-with-ansible/

**NOTE**

 Generate SSH key with SSH-keygen
 
  ssh-keygen
     
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
    es_ssl_truststore_password: "qwerty"
    es_ssl_verification_mode: certificate
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

After sucessful installation edit your elasticsearch.yml file(Add X-pack configuration)

```
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: "certificate"
xpack.security.transport.ssl.keystore.path: "/etc/elasticsearch/certs/http.p12"
xpack.security.transport.ssl.truststore.path: "/etc/elasticsearch/certs/http.p12"

xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: "/etc/elasticsearch/certs/http.p12"
xpack.security.http.ssl.truststore.path: "/etc/elasticsearch/certs/http.p12"
```


**Create a certificate to enable HTTPS communication**

Kindly follow the below guide. This is generated on the elasticsearch node copied to the control node

https://techexpert.tips/elasticsearch/elasticsearch-enable-tls-https/

    /usr/share/elasticsearch/bin/elasticsearch-certutil  http

Control Node(Ansible node)

    Path: /root/http.p12 (You may wish to use another path)

    
**Install Kibana manually**

     yum install Kibana

**NOTE** 

*Check the flavour of elasticseach you install, I had this issues when trying to start my kibana service*

**Create your SSL Certificate for communication to kibana over the browser**

Run this one either of the elasticsearch node to generate the SSL certificate

    /usr/share/elasticsearch/bin/elasticsearch-certutil ca --pem

**Create your PEM Certificate for communication between kibana and elasticsearch**

Run this 

      openssl pkcs12 -in elastic-certificates.p12 -cacerts -nokeys -out elasticsearch-ca.pem
      
**Configure Kibana.yml file**

```
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: 5601

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# The default is 'localhost', which usually means remote machines will not be able to connect.
# To allow connections from remote users, set this parameter to a non-loopback address.
server.host: "192.168.17.167"

# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["https://192.168.17.167:9200"]


# If your Elasticsearch is protected with basic authentication, these settings provide
# the username and password that the Kibana server uses to perform maintenance on the Kibana
# index at startup. Your Kibana users still need to authenticate with Elasticsearch, which
# is proxied through the Kibana server.
elasticsearch.username: "elastic"
elasticsearch.password: "qwerty"

# Enables SSL and paths to the PEM-format SSL certificate and SSL key files, respectively.
# These settings enable SSL for outgoing requests from the Kibana server to the browser.
server.ssl.enabled: true
server.ssl.certificate: /etc/kibana/ca/ca.crt
server.ssl.key: /etc/kibana/ca/ca.key

# Optional setting that enables you to specify a path to the PEM file for the certificate
# authority for your Elasticsearch instance.
elasticsearch.ssl.certificateAuthorities: [ "/etc/kibana/ca/elasticsearch-ca.pem" ]

# To disregard the validity of SSL certificates, change this setting's value to 'none'.
elasticsearch.ssl.verificationMode: none

```
 
 **Create the path where you want to store your certificates**
 
 ```
 [root@elk1 ~]# cd /etc/kibana/ca/
[root@elk1 ca]# ll
total 16
-rw-r--r--. 1 root root 1200 Aug 31 15:19 ca.crt
-rw-r--r--. 1 root root 1675 Aug 31 15:19 ca.key
-rw-r--r--. 1 root root 1367 Aug 31 15:43 ca.pem
-rw-r--r--. 1 root root 1397 Sep  2 13:49 elasticsearch-ca.pem
[root@elk1 ca]# pwd
/etc/kibana/ca
[root@elk1 ca]#

 ```

**Enable and start Elasticsearch**

     systemctl enable elasticsearch
     systemctl start elasticsearch

**Enable and start Kibana**

     systemctl enable kibana
     systemctl start kibana


 **And finally you have your Elasticsearch and Kibana Up and running**
 
 ![Elastic login Page](https://github.com/proxxious/ElasticStack-and-Ansible/blob/master/elasticlogin.JPG)
 

 
 
