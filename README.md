
          *--<< THIS PROJECT HAS BEEN CREATED AS PART OF 42 CURRICULUM BY FALAMLIH >>--*
  
`BORNTOBEROOT` is a project from 42 curriculum that allows student to learn how to setup a virtual machine on a server from partitions, security, permissions to install needed packages and setuping users

`Virtual machines` is the process of setuping multi os's on one physical machine using a tool called hypervisor (VBOX || UTM)

THERE IS 2 TYPES OF HYPERVISORS:
    `bare metal` : setuping the hypervisor on the physical machine directly
    `hosted` : setuping on OS already existed

THE PERPUSE OF VIRTUALISATION IS:
    testing multi OS's without affecting the main os of the pysical machine
    possibility of saving a backup of the os data with snapshots
    managing and controling the whole physical resources 

`VBOX` vs `UTM`
    UTM fromm Apple
    Vbox from Oracle
    utm doesn't have the option of snapshots natively in it UI, you should use QEMU program to make snapshos of vms in UTM
    vbox is user-friendly snapshot features in it UI, make it easier 
    UTM shines on Apple Silicon for ARM-based OSes, while VirtualBox has broader Linux support
    utm speed on ARM guests
    vbox speed on intel Macs
check graphic interface with : ls /usr/bin/*session

`ROCKY(CentOS)` vs `DEBIAN`:
    *DEBIAN -> General-purpose,ultimate freedom, user friendly and huge package selection
        why debian ?
        Debian is one of the oldest and most stable Linux distributions.
        ubuntu is based on debian, use APT as package manager, it is great for new linux users, for desktops and servers & has a huge software choice
        Debian does NOT enable SELinux by default
        There are many tutorials and resources available for Debian
        Pros:
            Stability & Reliability
            Vast Repositories
            Flexibility
        cons:
            Older Packages
            require more technical skill for initial setup

    *ROCKY -> Entreprise-grade, not 100% free, legacy apps, for business
        Red Hat(RHEL family) BASED
        Rocky Linux uses dnf and rpm  as package manager which are also powerful but less beginner-friendly for early Linux users.
        enable SELinux by default
        proc:
            Enterprise Focus
            Perfect for learning RHEL
            Desktop Friendly
            Stability
        cons:
            Smaller Package Set
            Less General Purpose
    Check OS: uname --kernel-version

`GRUB`  (Grand Unified Bootloader): 
    responsible for loading the operating system kernel into memory
    after the power on, the firmware(BIOS/ UEFI) initializes the hardware, then hands control tothe bootloader(GRUB)
    GRUB: Takes over, displays its menu (if configured), loads the chosen OS kernel, and passes control to the OS
    GRUB stands out among other bootloaders due to its versatility, feature-rich nature and can boot multiple operating systems

`APT` vs `APTITUDE` :
    they are package management tools for debian based systems
    *apt is the default Linux command-line tool to manage the packages on a Debian-based system, it is simpleer and installs packages fast without asking for options 
    *aptitude is more interactive, gives the users more options for managing packaging & solutions, It doesn't come by default, so you need to install it with the `apt` command
    ​Apt requires the user to have a solid knowledge of Linux systems and package management as you are running everything in the command line. It can be difficult for a novice to handle.ed9eea310dadce5d08d23020d676f56ca1a823fa
    On the other hand, aptitude with its interface is more user-friendly as it offers a layer of abstraction regarding the different sub-commands to use for installation, upgrades, etc.

To see the filesystem hierarchy : `man hier`

`LVM`: 
    logical volume manager is a linux storage managing, allow you to  create, resize and manage disk partitons while processing the system, and allows you to groupe several physical disk into one logical volume without losing data, also allows you to take snapshots(backups)
    mounting point is a directory in the file system where a partition or storage device can aceesed ,it provides a secure, encrypted connection between a client (our computer) and a server (a remote machine), protecting data
    lsblk

`SSH`:
    is a cryptographic network protocol used to securely operate network services over an unsecured network like the internet
     SSH is essential for accessing virtual machines. By using SSH, users can securely connect to their VMs, execute commands, and manage applications and services.
     it provides a secure pseudoterminal to user to have direct communication with a server
     IP adress is a unique identifier (adress) of every device connected to a network 
     An IPv4 address is a 32-bit number, typically written as four decimal numbers (octets) separated by dots, with each octet ranging from 0 to 255
     check ssh service :sudo service ssh status
     ssh newuser@localhost -p 4241

`AppArmor` VS `SELinux` :
    both are modules enhance Linux security through Mandatory Access Control (MAC)
        Mandatory access control (MAC) is a centralized access control system. MAC regulates access to resources based on the clearance levels of users and the attributes of objects they seek to access.
`AppArmor` is the default security module in Debian-based systems
    Easy to learn and manage
    Quick setup
    Uses human-readable	
    Static profiles require manual updates for app changes
    Not ideal for complex or dynamic environments
    A good fit for desktops and low-risk servers
`SELinux` is the default in Red Hat Enterprise Linux, CentOS and their derivatives
    Complex to configure and manage
    Enterprise-grade security
    Requires specialized tools
    Steep learning curve for new users
    
`UFW` vs `firewald`: only root can acess it /sbin/ufw
    A firewall is a network security device or software that monitors and controls incoming and outgoing network traffic based on predetermined security rules.
    Think of it as a barrier between your computer (or network) and the internet determining what data can enter or leave your computer or network
    *Firewalld provides a high-level interface for managing firewall rules and supports network zones, services, and rich rules. Firewalld is designed to be more user-friendly and is the default firewall solution in many Linux distributions, including RHEL/CentOS 
    *UFW (Uncomplicated Firewall) is a user-friendly interface for managing iptables rules on Ubuntu and Debian-based systems. It simplifies the process of configuring the firewall by providing easy-to-use command-line tools and a straightforward configuration syntax. UFW is aimed at simplifying firewall management for users who may not be familiar with the complexities of iptable.
    securing web serves / protecting ssh access / managing application acess & blocking unwanted traffic

    check the ufw service : sudo ufw satus \ sudo service ufw status 
    sudo ufw status numbered
    sudo ufw allow 8080
    sudo ufw delete num_rule
    

`USER` & `GROUP`:
    su :
       su allows commands to be run with a substitute user
      When called with no user specified, su defaults to running an
       interactive shell as root. When user is specified, additional
       arguments can be supplied, in which case they are passed to the
       shell without setting envirenement variables for the user or new shell
    (su -) :this also switch to another user but it loads full envirenement (default shell, env variables, paths...)
    adduser vs useradd :
        useradd is lowlevel command that needs to specify options manually like home directory or shell
        adduser is more user-friendly and high level command that provides by default the shell, home directory ...
    sudo adduser name_user
    sudo addgroup evaluating
    sudo adduser name_user evaluating
    changing hostname :
        sudo nano /etc/hostname 
        sudo nano /etc/hosts
        sudo reboot
        hostname
    changing the password:
        passwd 
    Check user and group:
        getent group sudo user42
    

`SUDO`:

    nano /etc/sudoers.d/sudo_config
    for the script to configure our sudo group
        -passwd_tries=3  ->  Limits incorrect password attempts
        -badpass_message= -> Custom error message
        -logfile, log_input, log_output -> Full auditing of sudo activity
        -requiretty -> Only allows sudo from a real terminal -> requiretty can prevent a    user from leaking their password in cleartext,TTY become required (real terminal session)Without a real TTY, attackers cannot trick system services or automated tasks into running sudo commands.
        -secure_path= -> Restricts what binaries sudo can execute
    Sudo -l to see it
    Sudo cannot be run by background scripts, automated tools, or non-interactive sessions.
   Check sudo : which sudo
   Check sudo logs :    
    cd /var/log/sudo
    ls
    cat sudo_config

`Password policy` :
first we've to modify on /etc/login.defs the max and min days
then installing the password quality library To enforce password quality rules : libpam-pwquality
    sudo apt install libpam-pwquality
then modifying on the /etc/pam.d/common-password :
    nano /etc/pam.d/common-password
retry=3
minlen=10 ➤ The minimum characters a password must contain.

ucredit=-1 ➤ The password must contain at least one capital letter. We must write it with a - sign, as this is how it knows that it refers to minimum characters; if we put a + sign it will refer to maximum characters.

dcredit=-1 ➤ The password must contain at least one digit.

lcredit=-1 ➤ The password must contain at least one lowercase letter.

maxrepeat=3 ➤ The password cannot have the same character repeated three consecutive times.

reject_username ➤ The password cannot contain the username within itself.

difok=7 ➤ The password must contain at least seven different characters from the last password used.

enforce_for_root ➤ We will implement this password policy for root.

`script` : squence of commands stored in one file that will be excuted and perform the function of each command
    >The grep command is a tool in Linux and Unix that's used to search for specific text within files, such as words, phrases or patterns
    >awk is a scripting language, and it is helpful when working in the command line. It's also a widely used command for text processing
    >the free command, a command-line tool showing system memory (RAM & Swap) usage, detailing total, used, free, shared, buffer/cache
    >the difference between print and printf in awk is print is Limited & automatically appends a newline character to the end of its output while printf is similar to the printf using in C lang, is uses format specifiers
    >vmstat command shows system statistics, allowing us to obtain a general detail of the processes, memory usage, CPU activity, system status, etc
    >The expr command in Linux is a command-line utility used to evaluate expressions and print the resulting value to standard output
    MAC Address is used to ensure the physical address of a computer.
    IP Address is the logical address of the computer.
    the journalctl command is a powerful Linux command-line utility used to view, filter, and manage system logs collected by the systemd journal
    The wall command in Linux (short for write all)
    sudo crontab -u root -e to edite on crontab file
    crontab: is a background process manager. The specified processes will be executed at the time you specify in the crontab file.
files used :
    /etc/ssh/sshd_config
    /etc/ssh/ssh_config
    /etc/sudoers.d/sudo_config
    /etc/login.defs
    /etc/pam.d/common-password
`Ressources` :
https://www.lenovo.com/us/en/glossary/grub/?orgRef=https%253A%252F%252Fwww.google.com%252F&srsltid=AfmBOormCgFQQkTEhz8HP2Jx2hsUiXCV9LWZcNntFND2873k4vX3rpov
https://fr.scribd.com/document/660125079/Linux-File-System-Ext2-vs-Ext3-vs-Ext4-vs-XFS
https://broman.dev/download/The%20Linux%20Programming%20Interface.pdf
https://born2beroot-tracking.pages.dev/
https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)
https://youtu.be/MeltFN-bXrQ
https://blog.packagecloud.io/know-the-difference-between-apt-and-aptitude/
https://amadla.medium.com/debian-linux-vs-rocky-os-exploring-the-best-choice-for-your-server-dfd6b3d80c1a
https://miro.com/app/board/uXjVLzBvyb8=/
https://tuxcare.com/blog/selinux-vs-apparmor/
https://nordlayer.com/learn/access-control/mandatory-access-control/
https://www.geeksforgeeks.org/linux-unix/difference-between-su-and-su-command-in-linux/
https://www.pluralsight.com/resources/blog/software-development/linux-firewall-administration


