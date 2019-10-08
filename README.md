# Ansible Mint setup

Ansible playbook whose aim is to save time setting up my Linux Mint XFCE machines.

## Testing

Before setting up a real machine I usually try this on up a [VirtualBox](https://www.virtualbox.org/). To easily create a new VM from scratch I use these commands.
(Heavily inspired by https://www.perkin.org.uk/posts/create-virtualbox-vm-from-the-command-line.html)

    ISO=/path/to/linux_mint.iso
    VM='LinuxMint'
    VBoxManage createvm --name $VM --ostype "Ubuntu_64" --register
    VBoxManage createhd --filename ~/'VirtualBox VMs'/$VM/$VM.vdi --size 32768
    VBoxManage storagectl $VM --name "SATA Controller" --add sata \
     --controller IntelAHCI
    VBoxManage storageattach $VM --storagectl "SATA Controller" --port 0 \
     --device 0 --type hdd --medium ~/'VirtualBox VMs'/$VM/$VM.vdi
    VBoxManage storagectl $VM --name "IDE Controller" --add ide
    VBoxManage storageattach $VM --storagectl "IDE Controller" --port 0 \
    --device 0 --type dvddrive --medium $ISO
    VBoxManage modifyvm $VM --ioapic on
    VBoxManage modifyvm $VM --boot1 dvd --boot2 disk --boot3 none --boot4 none
    VBoxManage modifyvm $VM --memory 4096 --vram 128
    # Port forwarding 2222 for ssh
    VBoxManage modifyvm $VM --natpf1 "guestssh,tcp,,2222,,22"
    # enable VNC server on port 3389
    VBoxManage modifyvm $VM --vrde on
    # set VNC password
    VBoxManage modifyvm $VM --vrdeproperty VNCPassword=secret

Now start the VM.

    VBoxManage startvm $VM

Or start headless and connect to localhost:3389 via vnc

    VBoxHeadless -s $VM

Run through the installer. Set up using the description below. Eject the "CD".

    VBoxManage storageattach $VM --storagectl "IDE Controller" --port 0 \
     --device 0 --type dvddrive --medium none

 Now it's time to take a snapshot.

    VBoxManage snapshot $VM take installer_finished

Try the ansible script. If you want to start from the previous state again run

    VBoxManage snapshot $VM restore installer_finished

## Manual steps while running the installer

#. Start _Install Linux Mint_
#. _English_
#. _German - German (no dead keys)_
#. _Install third-party_
#. _Erase disk_ plus _Encrypt_ plus _LVM_
#. Generate _Security Key_ in KeePass
#. _Berlin_
#. _Firstname_ _Lastname_
#. _Username_ `<myuser>`
