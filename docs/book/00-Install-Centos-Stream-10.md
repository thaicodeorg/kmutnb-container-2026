Install CentOS 10 Stream will pass through many step of configuration.
**Installation Step:**
1. After last step. we ready to install Cent OS 10 Stream as our server.  let Double Click on ``Virtual Machine name - CentOs 10 Stream`` . you will see pop up windows with start installing process of server
![Centos10Stream1.png](../assets/images/Centos10Stream1.png)

2. Use mouse to select popup windows to select Virtual Machine windows. then move menu up to ``Install CentOS Stream 10`` then click enter.  then we will enter to installation process.  
	2.1. Select Installation Language ``English`` . then Click ``Continue``
	![Centos10Stream2.png](../assets/images/Centos10Stream2.png)
	2.2. After Click continues, window pop up will show main configuration which need to setup correctly until button ``Begin Installation`` will enable and let we go to installation process. Let config installation.  Red notification text show configuration need to be attention
		- Installation Destination  will accept it default
		- Root Account   will set password to  'password'
		- User Account   will set  user is 'student' and password 'student'
	![Centos10Stream4.png](../assets/images/Centos10Stream4.png)
	2.3.  Select Installation Destination option by use mouse directly Click on ``Installation Destination``  In this step will show local hard disk for installation (in University lab may show 40 GB).  Select Storage Configuration at "Automatic" then Click "Done" to accept default
		![Centos10Stream5.png](../assets/images/Centos10Stream5.png)2.4. we move to setup Root user password. by use mouse click on ``Root Accont``
		![Centos10stream6.png](../assets/images/Centos10stream6.png)
		this step will set Root password. Select option ``Enable root account`` then you will see text box to set password as below.  We need to set root password  as 'password'  (can be more secure password)
		![Centos10Stream7.png](../assets/images/Centos10Stream7.png)
		2.5 After Click ``Done`` in 2.4 section already. installer will back to main configuration again. At final step is set second user which is "student" with password "student" this is the first student will have ``sudo`` privilege (use can use sudo command to run root command)
		![Centos10Stream8.png](../assets/images/Centos10Stream8.png)
		2.6. This user configuration is very simple. but there is special option to assign to student user.  When finish setup password, then click ``Done`` button.  this consider final setup.
		![Centos10Stream9.png](../assets/images/Centos10Stream9.png)2.7.  we will see ``Begin installation`` is enable that ready to install. Click button ``Begin installation`` to enter installation process
		![Centos10Stream10.png](../assets/images/Centos10Stream10.png)2.8.  Image below show installation process. Wait untill  finish installation process then restart Virtual Machine
		![Centos10Stream11.png](../assets/images/Centos10Stream11.png)2.9 Reboot
		![Centos10Stream12.png](../assets/images/Centos10Stream12.png)
3. Congratulation. we success to install Centos Stream 10. This version will also install Server GUI. we can use server direct to GUI or ssh from windows to Server. in Our Class we will simulate remote access to server like many system administrator do. 
	1. Open terminal in side Virtual machine
	2. type "command"  

```bash
$ cat /etc/os-release
$ ip a
```
login to server
![Centos10Stream13.png](../assets/images/Centos10Stream13.png)

Run ``cmd``   cat /etc/release with show server infomation 
![Centos10Stream14.png](../assets/images/Centos10Stream14.png)Run ``cmd``   ip  a  ( ip address) to get local ip address. this address we will use to ssh into server
![Centos10Stream15.png](../assets/images/Centos10Stream15.png)

From above command  server go dhcp ip address from hyper V is    ``172.27.231.186``  (student server will be not same)

4. Test Remote ssh from server ``cmd``  ssh student@172.27.231.186.  Image below show first time login. SSHD (openssh server) will need to ask for ``ED25519`` key exchange
![Centos10Stream16.png](../assets/images/Centos10Stream16.png)
but consequenctly login to server will not ask for key exchange process like terminal below
![Cent0s10Stream17.png](../assets/images/Cent0s10Stream17.png)

type password  "student". Done!  
![CentOs10Stream18.png](../assets/images/CentOs10Stream18.png)
Next Step We will install Docker in Cent OS Stream 10.  [ Docker Installation](02-installation.md) 