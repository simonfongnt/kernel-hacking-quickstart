# kernel-hacking-quickstart
[Kernel Hacking](https://kernelnewbies.org/) Quickstart
setup on Ubuntu machine

## Prerequisites:
To install all required packages on Ubuntu/Debian, run this command:
```
sudo apt-get install vim libncurses5-dev gcc make git exuberant-ctags libssl-dev bison flex libelf-dev bc dwarves zstd mutt git-email gitk linux-source python3-ply python3-git codespell 
```

<details close>
<summary> Tool Setups (Optional) </summary>

To configure the tools beforehand

<details close>    
  <summary>screen (good to have)</summary>
  
  On the machine for kernel hacking, run
  ```
  screen
  ```  
  Other machiens connected with ssh, run
  ```
  screen -x
  ```  
</details>
<details close>    
  <summary>vim</summary>
  
  Create a config file for vim  
  ```
  vim ~/.vimrc
  ```  
  Add the following lines:  
  ```
  filetype plugin indent on
  syntax on
  set title
  set tabstop=8
  set softtabstop=8
  set shiftwidth=8
  set noexpandtab
  ```  
  Make it as default editor  
  ```
  sudo update-alternatives --config editor
  ```  
  Then select `vim.basic` as default editor  
</details>
<details close>
  <summary>Email (Yahoo)</summary>
    
  Go click on your account icon (top right, above "Settings" and to the left of "Home"). Click on "Account info" and then go to "Account security". (You may have to sign in again for this step.) Scroll down to the setting "Allow apps that use less secure sign-in" and turn it on. If you have two-step verification or account key enabled, you will also need to use App Password. 
  
Amend the config file for mutt
```
vim ~/.muttrc
```
Add the following lines:
```
set envelope_from=yes
set from="REAL NAME <USERNAME@yahoo.com>"
set use_from=yes
set edit_headers=yes

set imap_user = 'USERNAME@yahoo.com'
set imap_pass = "CREATED_PASSKEY"
set header_cache=~/.mutt/cache/headers
set message_cachedir=~/.mutt/cache/bodies
set certificate_file=~/.mutt/certificates
set imap_keepalive = 300
set timeout = 15

set folder = "imaps://export.imap.mail.yahoo.com:993"
set spoolfile="imaps://imap.mail.yahoo.com/INBOX"
set postponed="imaps://imap.mail.yahoo.com/Drafts"
set record="imaps://imap.mail.yahoo.com/Sent"

set smtp_url = "smtp://USERNAME@smtp.mail.yahoo.com:587/"
set smtp_pass = "CREATED_PASSKEY"

set move = no
set sort = 'threads'
set sort_aux = 'last-date-received'
set imap_check_subscribed"

set mail_check = 90
```
</details>
<details>
<summary> Boot Menu (good to have) </summary>
  
Run
```
sudo vim /etc/default/grub
```
Comment the following lines:
```
# GRUB_HIDDEN_TIMEOUT=0
# GRUB_HIDDEN_TIMEOUT_QUIET=true
```
Adjust the following variables:
```
GRUB_TIMEOUT=10
GRUB_TIMEOUT_STYLE=menu
```
Apply changes
```
sudo update-grub2
```
</details>  
</details>

## Set up responsitory:
Run these two commands:

`mkdir -p git/kernels; cd git/kernels`

Then use the revision control system called git to clone Greg Kroah-Hartman's staging tree repository:

`git clone -b staging-testing git://git.kernel.org/pub/scm/linux/kernel/git/gregkh/staging.git`

Change into staging directory:
`cd staging`

## Config Kernel
As per a quickstart
```
cp /boot/config-`uname -r`* .config
make oldconfig
```
Upon changes
```
TBD...
```

## Build Kernel

Run
`make` or `sudo make`
Or, bet your luck with multi-core commands
```
lscpu     # check how many core the pc has
make -j`nproc`           # build the kernel
make -j`nproc` modules   # build the modules
```
### Issues?
- `install: setting permissions for ‘.../staging/tools/bpf/resolve_btfids/libbpf//include/bpf/bpf.h’: Operation not permitted`
  Run
  ```
  sudo make clean
  ```
  then make without sudo

- [`make[3]: *** No rule to make target 'debian/canonical-certs.pem', needed by 'certs/x509_certificate_list'.  Stop.`](https://stackoverflow.com/questions/67670169/compiling-kernel-gives-error-no-rule-to-make-target-debian-certs-debian-uefi-ce) OR
- [`make[3]: *** No rule to make target 'debian/canonical-revoked-certs.pem', needed by 'certs/x509_revocation_list'. Stop.`](https://stackoverflow.com/questions/67670169/compiling-kernel-gives-error-no-rule-to-make-target-debian-certs-debian-uefi-ce)
  
2 ways to bypass it:  
1. Disable the keys

Run
```
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS
```
2. or copy the keys from linux-buildinfo

Run
```
sudo apt install linux-buildinfo-$(uname -r)
sudo cp -v /usr/lib/linux/$(uname -r)/*.pem /usr/local/src/debian/

```
Update `.config` with these lines:
```
CONFIG_SYSTEM_TRUSTED_KEYS="/usr/local/src/debian/canonical-certs.pem"
CONFIG_SYSTEM_REVOCATION_KEYS="/usr/local/src/debian/canonical-revoked-certs.pem"
```
Run
```
make clean
make
```
  
## Install Kernel module (TBD)
Run
```
sudo make modules_install install
```

## Install Kernel
Run
```
sudo make modules_install # create dir of kernel modules for grub
sudo make install         # install kernel + modules + update grub
```
Reboot
```
sudo reboot
```
### Issues?
- `W: missing /lib/modules/x.xx.x-rc?+
W: Ensure all necessary drivers are built into the linux image!
depmod: ERROR: could not open directory /lib/modules/x.xx.x-rc?+: No such file or directory`
Modules was not built, run:
```
make -j`nproc` modules     # build the modules
sudo make modules_install  # create dir of kernel modules for grub
sudo make install          # install kernel + modules + update grub
```



## Undo (Be mindful to what is going to do below)
Clean up grub (i.e. anything under boot and named with rc)
```
ls /lib/modules/*rc* | sudo xargs -ixxx rm -rf 'xxx'
ls /boot/*rc* | sudo xargs -ixxx rm -rf 'xxx'
sudo update-grub
sudo reboot
```
