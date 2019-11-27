# okd-mixed-arch
 
## Prepare hosts - Centos 8

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
for i in `master01 master02 master03 worker01 worker02`;
do
ssh-copy-id -i ~/.ssh/id_rsa.pib $i
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

### Install Docker
```shell
for host in master01 master02 master03 worker01 worker02; do ssh -t $host 'yum -y install docker-1.13.1'; done
```


































# Disable ssh root login

