# Prepare WSL

To have a test environment for doing all exercises during the Community Call you can use *Windows Subsystem for Linx (WSL)*.

This guide shows you the basic steps to install **Ubuntu 24.04 LTS** in WSL, configured with *cloud-init*.  

## Create cloud-init configuration

Open the *Editor* App (or Notepad++ or any other editor) and paste in the following content:

> [!TIP]
> **Update the following cloud-init configuration!**  
> Adjust the **username**, after copying the content to a textfile, use search and replace!    
> Adjust the instance name, if necessary.

```yaml
#cloud-config
locale: C.UTF-8
hostname: wsl-ubuntu
users:
  - name: timgrt
    gecos: Tim Gruetzmacher
    groups: [adm,dialout,cdrom,floppy,sudo,audio,dip,video,plugdev,netdev]
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash

write_files:
  - path: /etc/wsl.conf
    append: true
    content: |
      [user]
      default=timgrt
      [network]
      generateResolvConf = false
      hostname = wsl-ubuntu
      generateHosts = false
  - path: /etc/inputrc
    content: |
      set bell-style visible
  
packages: [ginac-tools, octave, python3-pip, pipx, python3-venv, podman, dbus-user-session]

runcmd:
   - sudo git clone https://github.com/Microsoft/vcpkg.git /opt/vcpkg
   - sudo apt-get install zip curl -y
   - /opt/vcpkg/bootstrap-vcpkg.sh
   - git clone https://github.com/computacenter-com/ansible-community-call-exercises /home/timgrt/ansible-community-call-exercises
```

Save the file as `Ubuntu-24.04.user-data` in a new folder `.cloud-init` in your Windows Home directory (e.g. `C:\Users\tgruetz\.cloud-init\Ubuntu-24.04.user-data`). 

> [!WARNING]
> **Do not** save the file as a text file, change the filetype to *All files (*.*)*.

<img src=".assets/cloud-init-file.png" width=60%>

> [!IMPORTANT]
> ⚠️ The cloud-init config file **must** be called `Ubuntu-24.04.user-data` (the same as the distribution name), otherwise it won't be applied!

Take a look at the [Ubuntu on WSL installation with cloud-init documentation](https://documentation.ubuntu.com/wsl/latest/howto/cloud-init/) for further information.

## Install WSL distribution

Open a terminal (right-click the Start Button).  
Run the following command, it will install Ubuntu and configure it according to the cloud-init file:

```console
wsl --install Ubuntu-24.04
```

After installation is finished you will be logged in to Ubuntu. Close the WSL window or run `exit`. Open **Powershell** and restart WSL:

```console
wsl --terminate Ubuntu-24.04
```

Open a new terminal tab by clicking the ▼ symbol and choose your Ubuntu distribution.
