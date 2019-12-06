# okd-mixed-arch
 
## Prepare hosts - RHEL 7.7 

### Set /etc/hosts (if DNS is not available)
Include ALL nodes within the cluster


### Set PATH on all hosts within the Cluster
```shell
export PATH=$PATH:/bin/sbin
```

# Install SSH server
```shell
yum update -y
yum install openssh-server -y
systemctl start sshd
```


### Configure host access via passwordless ssh

master01
```shell
ssh-keygen # No Password
```

### May need to allow root login via ssh for duration of install.
### Centos 8 permits by default
```shell
PermitRootLogin Yes # /etc/ssh/sshd_config
```

Copy to all nodes within the Cluster
```shell
for host in `master01 master02 master03 worker01 worker02`;
do
ssh-copy-id -i ~/.ssh/id_rsa.pub $host
done
```

### Install Atomic for containerized install
```shell
for host in master01 master02 master03 worker01 worker02; do ssh -t $host 'yum install -y atomic'; done

```

### RPM Installer
```shell
for host in master01 master02 master03 worker01 worker02; do ssh -t $host 'yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm'; done
```

```shell
for host in master01 master02 master03 worker01 worker02; do ssh -t $host 'sed -i -e "s/^enabled=1/enabled=0/" /etc/yum.repos.d/epel.repo'; done
```

```shell
for host in master01 master02 master03 worker01 worker02; do ssh -t $host 'yum -y --enablerepo=epel install ansible pyOpenSSL'; done
```

### Enable Repos
subscription-manager repos --enable=rhel-7-server-rpms && \
subscription-manager repos --enable=rhel-7-server-extras-rpms && \
subscription-manager repos --enable=rhel-7-server-optional-rpms

yum install java-1.8.0-openjdk-headless -y

curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py

pip install passlib




### Install Docker
```shell
for host in master01 master02 master03 worker01 worker02; do ssh -t $host 'yum -y install docker-1.13.1'; done
```

# Create an OSEv3 group that contains the master, nodes, etcd, and lb groups.
# The lb group lets Ansible configure HAProxy as the load balancing solution.
# Comment lb out if your load balancer is pre-configured.
[OSEv3:children]
masters
nodes
etcd
#lb

# Set variables common for all OSEv3 hosts
[OSEv3:vars]
timeout=120
ansible_ssh_user=root
openshift_deployment_type=origin
os_firewall_use_firewalld=True
openshift_metrics_install_metrics=true
openshift_disable_check=memory_availability

# uncomment the following to enable htpasswd authentication; defaults to AllowAllPasswordIdentityProvider
#openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

# Native high availability cluster method with optional load balancer.
# If no lb group is defined installer assumes that a load balancer has
# been preconfigured. For installation the value of
# openshift_master_cluster_hostname must resolve to the load balancer
# or to one or all of the masters defined in the inventory if no load
# balancer is present.
openshift_master_cluster_method=native
openshift_master_cluster_hostname=kubevip01.home.swinney.io
openshift_master_cluster_public_hostname=kubevip01.home.swinney.io

# enable ntp on masters
openshift_clock_enabled=true

# host group for masters
[masters]
kubemaster01.home.swinney.io
kubemaster02.home.swinney.io
kubemaster03.home.swinney.io

# host group for etcd
[etcd]
kubemaster01.home.swinney.io
kubemaster02.home.swinney.io
kubemaster03.home.swinney.io

# Specify load balancer host
#[lb]
#kubevip01.home.swinney.io

# host group for nodes, includes region info
[nodes]
kubemaster0[1:3].home.swinney.io openshift_node_group_name='node-config-master-infra'
kubeworker0[1:3].home.swinney.io openshift_node_group_name='node-config-compute'
#kubeworker02.home.swinney.io openshift_node_group_name='node-config-compute'
#kubeworker03.home.swinney.io openshift_node_group_name='node-config-compute'



yum install java-1.8.0-openjdk-headless -y

































# Disable ssh root login

