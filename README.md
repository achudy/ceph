# Ceph repo for enginnering diploma
## AC

### Prerequisites

Install dependencies, set up storage for OSDs, set up ansible.

```
sudo su
useradd -d /home/cephuser -m cephuser
echo -e "cephuser\ncephuser" | passwd cephuser
echo "cephuser ALL = (root) NOPASSWD:ALL" | tee /etc/sudoers.d/cephuser
chmod 0440 /etc/sudoers.d/cephuser
su cephuser
```
```
cd
ssh-keygen
```

```
cat .ssh/id_rsa.pub 
vi .ssh/authorized_keys
chmod 700 .ssh/authorized_keys 
```

```
sudo yum install lvm2 -y
sudo vgcreate ceph-block-X /dev/sdb
sudo lvcreate -l 100%FREE -n osd-X ceph-block-X
sudo sed -i 's/python/python2/' /usr/bin/yum
sudo sed -i 's/python/python2/' /usr/libexec/urlgrabber-ext-down
```

Only on main node - one to execute ansible with.
```
sudo yum install ansible -y
sudo yum  install python-netaddr -y
sudo yum install git -y
git clone https://github.com/guv4/ceph.git
cd ceph/
```
Edit cluster IPs and OSD storage devices:
```
# cd container
# cd baremetal
vi inventory/hosts
```


### Execution
For baremetal:
```
ansible-playbook -i baremetal/inventory/hosts site.yml -vv
```
For container:
```
ansible-playbook -i container/inventory/hosts site-container.yml -vv
```

### Mounting the data to a client folder
```
mkdir /mount-point
mount -t ceph IP_1,IP_2,IP_3,...:/ /mount-point 
```


### Tests
Install HDPARM
```
sudo yum install -y hdparm
```

Make a test pool for the benchmarks.
```
ceph osd pool create scbench 16 16
```

Write and execute such script to get results in text form.

This should be done with access to ceph (in ceph-mon container or on the main node with baremetal).
`hdparm` should't be executed in a container, because it is a hadware test.
```
#/bin/bash
for i in {1..5}
do
    hdparm -tT /dev/sdb >> results_hdparm.txt
done

for i in {1..10}
do
    rados bench -p scbench 10 write --no-cleanup >> results_wr.txt
done

for i in {1..10}
do
    rados bench -p scbench 10 seq >> results_wr.txt
done

for i in {1..10}
do
    rados bench -p scbench 10 rand >> results_wr.txt
done
```
