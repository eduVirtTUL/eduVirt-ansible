# EduVirt environment

## Running the playbooks

### Execution environment

Execution environment provides a portable environment for running the playbooks. It uses a container to store all neccessary python dependiencies and ansible collections.    

### Building the execution environment 

To build a local execution environment run the following command.

```
ansible-builder build --tag eduvirt_ee --container-runtime docker -f execution_environment/execution-environment.yaml
```

### Using the execution environment 

Execution environment can be used to lauch playbooks using ansible-navigator. For example: 

```
ansible-navigator run setup_ovirt_playbook.yaml -i inventories/set2/hosts.yaml  --execution-environment-image eduvirt_ee --pull-policy missing --mode stdout
```

Specified pull policy is important, as it prioritizes the locally built image. Selected mode makes the output format the same as running a `ansible-playbook`.

### Alternative 

If you don't want to use docker to run playbooks, or would like to develop the playbooks using something like Ansible VSCode extension, you can setup a local environment.


In this case you need to create a python virtual environment using the `requirements.txt` file in the root of the project. After that you should install the packages below.

```
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install ovirt.ovirt
ansible-galaxy collection install community.general
ansible-galaxy install ovirt.ovirt-ansible-roles
```

This allows to run the playbook this way:

```
ansible-playbook -i inventories/set2/hosts.yaml setup_ovirt_playbook.yaml
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