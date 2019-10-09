# Ansible Mint setup

Ansible playbook whose aim is to save time setting up my Linux Mint XFCE machines.

## Ansible installation

Setup ansible on the *host* (not VM guest).

    sudo apt update
    sudo apt install -y python3-venv
    python3 -m venv ~/Software/ansible
    . ~/Software/ansible/bin/activate
    pip install wheel
    sudo apt install -y libssl-dev python3-dev
    pip install ansible==2.8.5
    deactivate
    mkdir -p ~/bin
    ln -s ~/Software/ansible/bin/ansible-playbook ~/bin/
    ln -s ~/Software/ansible/bin/ansible ~/bin/

## Testing

Before setting up a real machine I usually try this on up a [VirtualBox](https://www.virtualbox.org/) (see below). Once set up I can test the ansible playbook.

    ansible-playbook playbook.yaml -i vbox -K

To easily create a new VM from scratch I use these commands. (Heavily inspired by https://www.perkin.org.uk/posts/create-virtualbox-vm-from-the-command-line.html)

    ISO=/path/to/linux_mint.iso
    VM='LinuxMint'
    MEDIUM="$HOME/VirtualBox VMs/$VM/$VM.vdi"
    VBoxManage createvm --name $VM --ostype "Ubuntu_64" --register
    VBoxManage createhd --filename "$MEDIUM" --size 32768
    VBoxManage storagectl $VM --name "SATA Controller" --add sata \
     --controller IntelAHCI
    VBoxManage storageattach $VM --storagectl "SATA Controller" --port 0 \
     --device 0 --type hdd --medium "$MEDIUM"
    VBoxManage storagectl $VM --name "IDE Controller" --add ide
    VBoxManage storageattach $VM --storagectl "IDE Controller" --port 0 \
     --device 0 --type dvddrive --medium $ISO
    VBoxManage modifyvm $VM --ioapic on
    VBoxManage modifyvm $VM --boot1 dvd --boot2 disk --boot3 none --boot4 none
    VBoxManage modifyvm $VM --memory 4096 --vram 128
    # Port forwarding 3222 for ssh
    VBoxManage modifyvm $VM --natpf1 "guestssh,tcp,,3222,,22"

Now start the VM.

    VBoxManage startvm $VM

Or start headless and connect to localhost:3389 via vnc

    # enable VNC server on port 3389
    VBoxManage modifyvm $VM --vrde on
    # set VNC password
    VBoxManage modifyvm $VM --vrdeproperty VNCPassword=secret
    VBoxHeadless -s $VM

Run through the installer. Set up using the description below.

Setup SSH server on the VM.

    sudo apt install -y openssh-server

Setup SSH authorized keys and configure `vbox` alias.

```
ssh-copy-id -p 3222 -i ~/.ssh/id_rsa localhost
cat >> ~/.ssh/config <<HERE
Host vbox
  HostName localhost
  Port 3222
HERE
```

Install the guest additions.

    VBoxManage storageattach $VM --storagectl "IDE Controller" --port 0 \
     --device 0 --type dvddrive --medium /usr/share/virtualbox/VBoxGuestAdditions.iso
    ssh vbox
    user@vbox$ sudo mount /dev/cdrom /media && sudo /media/VBoxLinuxAdditions.run
    user@vbox$ sudo shutdown -h now

Now it's time to take a snapshot.

    VBoxManage snapshot $VM take installer_finished

Try the ansible script. If you want to start from the previous state again run

    VBoxManage snapshot $VM restore installer_finished

## Manual steps while running the installer

1. Start _Install Linux Mint_
1. _English_
1. _German - German (no dead keys)_
1. _Install third-party_
1. _Erase disk_ plus _Encrypt_ plus _LVM_
1. Generate _Security Key_ in KeePass
1. _Berlin_
1. _Name_ `<Firstname> <Lastname>`
1. _Computer's name_ `vbox`
1. _Username_ `<myuser>`
1. _Require login_
