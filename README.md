# chroot_sshd
Setup script for chroot jailing ssh and sftp via open-ssh

This script is intended to assist in creating chroot jails for ssh (and
by extension, sftp) users on a system. 

### OPERATION

_createjail JAILPATH OPTPROGS[...]_

*JAILPATH* is the full path of the jail environment you want to create.

*OPTPROGS* are the optional programs you want available. The full path
to the program is required.

### EXAMPLES
  
  _createjail /tmp/jail /bin/bash_ 

This will create an environment only providing the bash login shell (and
no other programs) to the user.

 _createjail /tmp/jail /bin/bash /bin/cp /bin/mv /bin/ls /bin/mkdir
/usr/bin/clear_

This will create an envrionment where a user can log in and do basic
file operations and clear the screen.  They will be prohibited from
deleting anything though.

### BASIC DIRECTORIES

During execution, the script will create those directories necessary for
basic system operation -- bin, dev, etc, home, lib, lib64, usr, and
usr/bin -- as well as the JAILPATH, if required.  Additional directories
under /lib and /lib64 will be created based on any library dependencies
of the program(s) you choose to install.


### MANUAL OPERATIONS

As it stands right now, this script will only create the directory
structure/permissions required for the chroot jail, and also optionally
create a group for jailed users.

You will still need to do the following:

1. Create new users for the jail / add users to the jailed group
  1. Create a new user with  
     `sudo adduser --home /tmp/jail/home/username username`  
     and then edit `/etc/passwd` to have their home set as `/home/user`
  2. Move an existing user's `/home/user` directory to `/tmp/jail/home` 
2. Update your /etc/ssh/sshd_config file to enable the jail.
  * Example, assuming we're using the group "jailed" :
  
  Match Group jailed  
    ChrootDirectory /tmp/jail  
    X11Forwarding no  
    AllowTcpForwarding no  
    AuthorizedKeysFile /tmp/jail/home/%u/.ssh/authorized_keys
  
  * NOTE -- if you want the group to only be allowed sftp access, add
     the directive ForceCommand internal-sftp to the above list.  

3. Restart the ssh service.
