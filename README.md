# vscode-remote-development-in-colab

1.First you need to install vscode remote development extension

`https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack`

 For windows local mechine
 ---
 
 1. open powershell in .ssh folder in windows and run
 
 `ssh-keygen -t rsa -b 4096 -f debian_rsa`
 
 this will generate "debian_rsa" named public and private  key
 
 ![sf](https://user-images.githubusercontent.com/11449967/62817862-9b1a7380-bb5f-11e9-9783-60cd6ccc16ad.PNG)
 
![wd](https://user-images.githubusercontent.com/11449967/62817895-23007d80-bb60-11e9-9831-00c2019eef7d.PNG)

2. Now got to the google colab and run this code

```python
import random, string, urllib.request, json, getpass

#Generate root password
password = ''.join(random.choice(string.ascii_letters + string.digits) for i in range(20))

#Download ngrok
! wget -q -c -nc https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
! unzip -qq -n ngrok-stable-linux-amd64.zip

#Setup sshd
! apt-get install -qq -o=Dpkg::Use-Pty=0 openssh-server pwgen > /dev/null

#Set root password
! echo root:$password | chpasswd
! mkdir -p /var/run/sshd
! echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
! echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
! echo "LD_LIBRARY_PATH=/usr/lib64-nvidia" >> /root/.bashrc
! echo "export LD_LIBRARY_PATH" >> /root/.bashrc

#Run sshd
get_ipython().system_raw('/usr/sbin/sshd -D &')

#Ask token
print("Copy authtoken from https://dashboard.ngrok.com/auth")
authtoken = getpass.getpass()

#Create tunnel
get_ipython().system_raw('./ngrok authtoken $authtoken && ./ngrok tcp 22 &')

#Get public address and print connect command
with urllib.request.urlopen('http://localhost:4040/api/tunnels') as response:
  data = json.loads(response.read().decode())
  (host, port) = data['tunnels'][0]['public_url'][6:].split(':')
  print(f'SSH command: ssh -p{port} root@{host}')

#Print root password
print(f'Root password: {password}')
```

2.Now go to Ngrok website login  and Copy authtoken from https://dashboard.ngrok.com/auth

3.Now put the tocken of ngrok in to the console of colab.

4.Colab will generate the user id and password for ssh like 

```
   SSH command: ssh -pxxx root@0.tcp.ngrok.io
   Root password: xxxxxxxxxxxxxxx
 ```

5.Now copy your public key file from .shh folder an put it on ` /usr/.ssh/authorized_key/` folder if it does not exist just do mkdir to create the directory.

![dwd](https://user-images.githubusercontent.com/11449967/62817986-e2a1ff00-bb61-11e9-9356-1379edca95ab.PNG)

6.Now come back to your windows local mechine and go to .ssh folder again and run

`ssh -i [your_private_key_name] -p[port] root@0.tcp.ngrok.io`

Ex: `ssh -i debian_rsa -p16542 root@0.tcp.ngrok.io`

7.Now open vscode in local windows mechine and make config file for vscode remote development like this 

```
Host colab
    HostName 0.tcp.ngrok.io
    User root
    Port 16792
    IdentityFile C:/Users/.ssh/debian_rsa
```
8. Press connect on vscode remote extension and that's it .. .
