## Creating Templates and Virtual Machines on Proxmox using Rocky Cloud Images

```bash
sudo apt update && sudo apt install libguestfs-tools
export IMAGES_PATH="/mnt/pve/nfs-data/images/"
mkdir -p ${IMAGES_PATH}
cd ${IMAGES_PATH}
wget https://dl.rockylinux.org/vault/rocky/9.3/images/x86_64/Rocky-9-GenericCloud-Base.latest.x86_64.qcow2
IMAGE="Rocky-9-GenericCloud-Base.latest.x86_64.qcow2"
TID=9999
TNAME=rocky9-cloudinit-template
CUSER=root
SIZE=10G
BRIDGE=vmbr2
RAM=2048
CORES=1
STORAGE=ssd
CPASS="xxxx"
virt-customize -a $IMAGE --install vim,mtr,tar,curl,wget,lsof,unzip,lrzsz,rsync,telnet,bash-completion,nmap-ncat,yum-utils,nfs-utils,iproute,util-linux,libseccomp,libseccomp-devel,qemu-guest-agent
virt-customize -a $IMAGE --run-command 'systemctl enable qemu-guest-agent'
virt-customize -a $IMAGE --timezone "Asia/Ho_Chi_Minh"
virt-customize -a $IMAGE --run-command 'sed -i "s/.*PermitRootLogin.*/PermitRootLogin yes/g"  /etc/ssh/sshd_config'
virt-customize -a $IMAGE --run-command 'sed -i "s/.*PubkeyAuthentication.*/PubkeyAuthentication yes/g"  /etc/ssh/sshd_config'
virt-customize -a $IMAGE --selinux-relabel
virt-customize -a $IMAGE --run-command 'sed -i "s/^SELINUX=.*/SELINUX=disabled/g" /etc/selinux/config'
virt-customize -a $IMAGE --run-command 'rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org'
virt-customize -a $IMAGE --run-command 'dnf install https://www.elrepo.org/elrepo-release-9.el9.elrepo.noarch.rpm -y'
virt-customize -a $IMAGE --run-command 'dnf --enablerepo=elrepo-kernel install kernel-lt -y'
virt-customize -a $IMAGE --run-command 'dnf --enablerepo=elrepo-kernel install kernel-lt-modules-extra -y'
qm create $TID --memory $RAM --cores $CORES  --net0 virtio,bridge=$BRIDGE --scsihw virtio-scsi-pci
qm importdisk $TID $IMAGE $STORAGE
qm set $TID --ide0 media=cdrom,file=none
qm set $TID --scsihw virtio-scsi-pci --scsi0 $STORAGE:vm-$TID-disk-0
qm set $TID --agent 1
qm set $TID --ide2 $STORAGE:cloudinit --boot order=scsi0
qm set $TID --ipconfig0 ip=dhcp --cipassword="$CPASS" --ciuser=$CUSER
qemu-img resize $IMAGE $SIZE
echo $TNAME
qm set $TID --name $TNAME
qm template $TID
```
