# Ansible Mint setup

![CI](https://github.com/baztian/ansible-mint-setup/workflows/CI/badge.svg)

Ansible playbook whose aim is to save time setting up my Linux Mint XFCE machines.

## Ansible installation

Setup ansible on the *host* (not VM guest).

    sudo -s
    apt update
    apt install -y python3-venv libssl-dev python3-dev
    python3 -m venv /opt/ansible-2.8.5
    . /opt/ansible-2.8.5/bin/activate
    pip install wheel
    pip install ansible==2.8.5
    deactivate
    ln -sf /opt/ansible-2.8.5/bin/ansible-playbook /usr/local/bin/
    exit

## Usage

Install on local machine using `ansible-pull`.

    ansible-pull -U https://github.com/baztian/ansible-mint-setup.git -i local playbook.yml -K

Install on local machine using stock `ansible`.

    ansible-playbook playbook.yml -i local -K

Install but skip some steps.

    ansible-playbook playbook.yml -i local -K --skip-tags xfce,calibre,eac,spotify,multimedia,squeezeplay

Force upgrade required roles.

    ansible-galaxy install -f -r requirements.yml

## Manual steps while running the Linux Mint installer

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

## Testing

Before setting up a real machine I usually try this on up a [VirtualBox](https://www.virtualbox.org/) (see below). Once set up I can test the ansible playbook.

    ansible-playbook playbook.yml -i vbox -K

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
ssh-copy-id -p 3222 -o IdentitiesOnly=yes -i ~/.ssh/id_rsa localhost
cat >> ~/.ssh/config <<HERE
Host vbox
  HostName localhost
  Port 3222
  IdentityFile ~/.ssh/id_rsa.pub
  IdentitiesOnly yes
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

## Customization and development notes

The following notes might help you in developing and customizing the roles.

### Identifying xfconf properties

To browse Xfce configuration use `xfce4-settings-editor`.

You can monitor so called channels from the command line with the `-m` switch of `xfconf-query`.

    xfconf-query # list all channels available
    xfconf-query -m -c xfce4-desktop -v

To monitor all available channels use this one liner.

    for i in $(xfconf-query); do (xfconf-query -m -c $i -v |awk '{print "'$i' " $0}' &) ;done

You can cancel monitoring by issuing

    killall xfconf-query

## Role dependencies

This playbook depends on several roles.

|Galaxy Role|Github Role|
| --------- | --------- |
| [baztian.apt_upgrade](https://galaxy.ansible.com/baztian/apt_upgrade) | [![CI](https://github.com/baztian/ansible-apt-upgrade/workflows/CI/badge.svg)](https://github.com/baztian/ansible-apt-upgrade) |
| [baztian.git](https://galaxy.ansible.com/baztian/git) | [![CI](https://github.com/baztian/ansible-git/workflows/CI/badge.svg)](https://github.com/baztian/ansible-git) |
| [baztian.asdf](https://galaxy.ansible.com/baztian/asdf) | [![CI](https://github.com/baztian/ansible-asdf/workflows/CI/badge.svg)](https://github.com/baztian/ansible-asdf) |
| [baztian.docker](https://galaxy.ansible.com/baztian/docker) | [![CI](https://github.com/baztian/ansible-docker/workflows/CI/badge.svg)](https://github.com/baztian/ansible-docker) |
| [baztian.vscode](https://galaxy.ansible.com/baztian/vscode) | [![CI](https://github.com/baztian/ansible-vscode/workflows/CI/badge.svg)](https://github.com/baztian/ansible-vscode) |
| [baztian.python](https://galaxy.ansible.com/baztian/python) | [![CI](https://github.com/baztian/ansible-python/workflows/CI/badge.svg)](https://github.com/baztian/ansible-python) |
| [baztian.pip_venv](https://galaxy.ansible.com/baztian/pip_venv) | [![CI](https://github.com/baztian/ansible-pip-venv/workflows/CI/badge.svg)](https://github.com/baztian/ansible-pip-venv) |
| [baztian.adr_tools](https://galaxy.ansible.com/baztian/adr_tools) | [![CI](https://github.com/baztian/ansible-adr-tools/workflows/CI/badge.svg)](https://github.com/baztian/ansible-adr-tools) |
| [baztian.dev_tools](https://galaxy.ansible.com/baztian/dev_tools) | [![CI](https://github.com/baztian/ansible-dev-tools/workflows/CI/badge.svg)](https://github.com/baztian/ansible-dev-tools) |
| [baztian.java](https://galaxy.ansible.com/baztian/java) | [![CI](https://github.com/baztian/ansible-java/workflows/CI/badge.svg)](https://github.com/baztian/ansible-java) |
| [baztian.aws](https://galaxy.ansible.com/baztian/aws) | [![CI](https://github.com/baztian/ansible-aws/workflows/CI/badge.svg)](https://github.com/baztian/ansible-aws) |
| [baztian.keepass](https://galaxy.ansible.com/baztian/keepass) | [![CI](https://github.com/baztian/ansible-keepass/workflows/CI/badge.svg)](https://github.com/baztian/ansible-keepass) |
| [baztian.calibre](https://galaxy.ansible.com/baztian/calibre) | [![CI](https://github.com/baztian/ansible-calibre/workflows/CI/badge.svg)](https://github.com/baztian/ansible-calibre) |
| [baztian.joplin](https://galaxy.ansible.com/baztian/joplin) | [![CI](https://github.com/baztian/ansible-joplin/workflows/CI/badge.svg)](https://github.com/baztian/ansible-joplin) |
| [baztian.gocryptfs](https://galaxy.ansible.com/baztian/gocryptfs) | [![CI](https://github.com/baztian/ansible-gocryptfs/workflows/CI/badge.svg)](https://github.com/baztian/ansible-gocryptfs) |
| [baztian.emacs](https://galaxy.ansible.com/baztian/emacs) | [![CI](https://github.com/baztian/ansible-emacs/workflows/CI/badge.svg)](https://github.com/baztian/ansible-emacs) |
| [baztian.jetbrains_toolbox](https://galaxy.ansible.com/baztian/jetbrains_toolbox) | [![CI](https://github.com/baztian/ansible-jetbrains-toolbox/workflows/CI/badge.svg)](https://github.com/baztian/ansible-jetbrains-toolbox) |
| [baztian.js](https://galaxy.ansible.com/baztian/js) | [![CI](https://github.com/baztian/ansible-js/workflows/CI/badge.svg)](https://github.com/baztian/ansible-js) |
| [baztian.jython](https://galaxy.ansible.com/baztian/jython) | [![CI](https://github.com/baztian/ansible-jython/workflows/CI/badge.svg)](https://github.com/baztian/ansible-jython) |
| [baztian.other_installs](https://galaxy.ansible.com/baztian/other_installs) | [![CI](https://github.com/baztian/ansible-other-installs/workflows/CI/badge.svg)](https://github.com/baztian/ansible-other-installs) |
| [baztian.multimedia](https://galaxy.ansible.com/baztian/multimedia) | [![CI](https://github.com/baztian/ansible-multimedia/workflows/CI/badge.svg)](https://github.com/baztian/ansible-multimedia) |
| [baztian.squeezeplay](https://galaxy.ansible.com/baztian/squeezeplay) | [![CI](https://github.com/baztian/ansible-squeezeplay/workflows/CI/badge.svg)](https://github.com/baztian/ansible-squeezeplay) |
| [baztian.skype](https://galaxy.ansible.com/baztian/skype) | [![CI](https://github.com/baztian/ansible-skype/workflows/CI/badge.svg)](https://github.com/baztian/ansible-skype) |
| [baztian.xfce](https://galaxy.ansible.com/baztian/xfce) | [![CI](https://github.com/baztian/ansible-xfce/workflows/CI/badge.svg)](https://github.com/baztian/ansible-xfce) |
| [baztian.signal](https://galaxy.ansible.com/baztian/signal) | [![CI](https://github.com/baztian/ansible-signal/workflows/CI/badge.svg)](https://github.com/baztian/ansible-signal) |
| [baztian.vagrant](https://galaxy.ansible.com/baztian/vagrant) | [![CI](https://github.com/baztian/ansible-vagrant/workflows/CI/badge.svg)](https://github.com/baztian/ansible-vagrant) |
| [baztian.eac](https://galaxy.ansible.com/baztian/eac) | [![CI](https://github.com/baztian/ansible-eac/workflows/CI/badge.svg)](https://github.com/baztian/ansible-eac) |
