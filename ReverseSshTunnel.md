#Reverse SSH Tunnel for Raspberry pi

##Source:

##[http://www.tunnelsup.com/raspberry-pi-phoning-home-using-a-reverse-remote-ssh-tunnel](http://www.tunnelsup.com/raspberry-pi-phoning-home-using-a-reverse-remote-ssh-tunnel)

* * * *

cd ~/.ssh

ssh-keygen -t rsa #no password and default file name

vim config

~~~

Host LinuxServer

 Hostname #IP/Hostname of Linux Server
 
 IdentityFile ~/.ssh/id_rsa
 
 user #User on Linux Server
 
 Port #Port on Linux Server for OpenSSH Server
 
 TCPKeepAlive yes
 
 #ForwardX11Trusted yes #only if neaded
 
~~~
 
scp id_rsa.pub LinuxServer:.ssh/authorized_keys

vim tunnel.sh

~~~

#!/bin/bash

createTunnel() {
  
  /usr/bin/ssh -N -R 2222:localhost:22 LinuxServer //if OpenSSH server port for the Pi is 22
  
  if [[ $? -eq 0 ]]; then
    
    echo Tunnel to jumpbox created successfully
  
  else
    
    echo An error occurred creating a tunnel to jumpbox. RC was $?
 
  fi

}

/bin/pidof ssh

if [[ $? -ne 0 ]]; then
  
  echo Creating new tunnel connection
  
  createTunnel

fi

~~~

crontab -e

~~~

#Make the raspberry pi connect back to a Linux Server every day between 20 and 21.

#On the Linux server run "ssh -l PiUser -p 2222 localhost" 

*/1 20 * * * /home/pi/ssh_tunnel.sh > tunnel.log 2>&1

~~~