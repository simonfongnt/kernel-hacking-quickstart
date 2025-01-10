# kernel-hacking-quickstart
Kernel Hacking Quickstart
setup on Ubuntu machine

## Prerequisites:
To install all required packages on Ubuntu/Debian, run this command:
```
sudo apt-get install vim libncurses5-dev gcc make git exuberant-ctags libssl-dev bison flex libelf-dev bc dwarves zstd mutt git-email gitk linux-source
```

<details close>
<summary> Tool Setups (Optional) </summary>

To configure the tools beforehand
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
make -j8   # 4 core to be used
```
Issues?
- `install: setting permissions for ‘.../staging/tools/bpf/resolve_btfids/libbpf//include/bpf/bpf.h’: Operation not permitted`
  ```
  sudo make
  ```


- [`make[3]: *** No rule to make target 'debian/canonical-revoked-certs.pem', needed by 'certs/x509_revocation_list'. Stop.`](https://stackoverflow.com/questions/67670169/compiling-kernel-gives-error-no-rule-to-make-target-debian-certs-debian-uefi-ce)

  Run
  ```
  sudo mkdir -p /usr/local/src/debian
  sudo apt install linux-source
  sudo cp -v /usr/src/linux-source-*/debian/canonical-*.pem /usr/local/src/debian/  
  ```
  Update `.config` with this line:
  ```
  CONFIG_SYSTEM_REVOCATION_KEYS="/usr/local/src/debian/canonical-revoked-certs.pem"
  ```
  Run
  ```
  sudo make
  ```
  
## Install Kernel module (TBD)
Run
```
sudo make modules_install install
```

## Install Kernel
Run
```
sudo make install
```
Reboot
```
sudo reboot
```
