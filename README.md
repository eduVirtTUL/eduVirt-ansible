# EduVirt environment

## Before running

### Inventory

Create your inventory that will be used to install the ovirt on.
Example inventory is provided in the `inventories/example` directory. 

### SSH keys

Ansible connects to the managed hosts using ssh. Because of that you need to provide it with ssh keys used to establishing the connections.

Create a `ssh_keys` directory in the root of the project. There, add all keys you need to connecting to the hosts. 

In the example inventory all hosts will try to use the key specified in `ovirt_machines.yaml` file. If hosts use different keys, specify them by overriding the `ansible_ssh_private_key_file` variable in a respective node file.

### Secret management

Ansible provides a mechanism called *ansible-vault* that allows to encrypt sensitive information and use it easily in the playbooks. It uses AES256 to encrypt the files or variables, which potentially allows to share them publicly. 

If you decide to use it create a file and change the `vault_password_file` to that file. Now you can use the *ansible-vault* without providing passwords on all playbook invocations.

For more advanced usage see https://docs.ansible.com/ansible/latest/vault_guide/index.html.

## Running the playbooks

### Execution environment

Execution environment provides a portable environment for running the playbooks. It uses a container to store all neccessary python dependiencies and ansible collections.    

### Using the execution environment

Create a python virtual environment and activate it.
```
python3 -m venv .venv
source .venv/bin/activate
```

Install `ansible-navigator` to run playbooks on the container:
```
python3 -m pip install ansible-navigator
```

Execution environment can be used to lauch playbooks using ansible-navigator. For example: 

```
ansible-navigator run test_ee.yaml -i inventories/set2/hosts.yaml
```

### Alternative 

If you don't want to use docker to run playbooks, or would like to develop the playbooks using something like Ansible VSCode extension, you can setup a local environment.


**You need at least python 3.11 for this to work.**

In this case you need to create a python virtual environment and install packaged using the `requirements.txt` file in the root of the project. 
```
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```


After that you should install the packages below.

```
ansible-galaxy collection install ansible.posix ovirt.ovirt community.general 
ansible-galaxy install ovirt.ovirt-ansible-roles
```

This allows to run the playbook this way:

```
ansible-playbook -i inventories/set2/hosts.yaml test_ee.yaml
```

## Create kickstart disk

Create raw virtual disk using qemu-img.
```bash
qemu-img create -f raw <disk_name> <disk_size>
```

Attach newly created disk to a loop device. (If needed create it before it)
```bash
losetup /dev/loop0 <disk_name>
```

Partition the disk to prepare it for filesystem creation. Accept the defaults.
```bash
fdisk /dev/loop0
```

Create file system and label the disk as ```OEMDRV```. This causes the disk to be automatically searched as a possible location for the kickstart file. During filesystem creation.
```bash
mkfs.ext4 -L OEMDRV /dev/loop0
```

Confirm the label was properly created.
```bash
e2label /dev/loop1
```

Mount the newly formated disk. (Create mount location if necessary)
```bash
mount /dev/loop0 /mnt/disk
```

Copy prepared beforehand kickstart file. Red Hat provides a basic configuration tool at https://access.redhat.com/labs/kickstartconfig/ accessible with free developer subscription.
```bash
cp ks.cfg /mnt/disk/
``` 

Unount the the disk and detach it from loop device.
```bash
umount /dev/loop0 /mnt/disk
losetup -d /dev/loop0
```

The disk is ready to be uploaded to the Ovirt Engine. Format of the disk should be specified as raw.

---
**Note**

The packages chosen in the kickstart file  need to be provided in the selected iso image, otherwise the installator will wait for manual confirmation of installation without required packages.

--- 