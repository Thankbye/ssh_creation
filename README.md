# The SSH Protocol

## Introduction

**SSH**, or Secure Shell, is a cryptographic network protocol used to securely access and manage devices over an unsecured network. It provides strong authentication and encrypted data communications, making it ideal for remote administration and file transfers.

SSH replaces **FTP** (File Transfer Protocol) because it offers enhanced security features. While FTP transmits data in plain text, making it vulnerable to eavesdropping, SSH encrypts the entire session, protecting sensitive information. Additionally, SSH supports secure file transfers through protocols like SFTP (SSH File Transfer Protocol), making it a more secure and versatile choice for managing files and systems remotely.

## Setting Up

To connect to an existant SSH session we need to use this command : 

> ssh user@IP

When you type this SSH will ask you if you want to add this to **know_hosts**, it ask you this to store the IP of the server, this will ensure a more secure session. After that it will ask you your password then you will be connected

## SSH Keys

#### Generate the keys

We can generate a pair of SSH keys to secure way more our session, in the pair there is a priviate keys (**id_rsa**) and a public key (**id_rsa.pub**). We can create the key with this:

> ssh-keygen -t ed25519

**ed25519** is an algorithm we can use another if you know or prefer other. After doing this we send the public key to the SSH session, it is essential so the private key can identify to the public key, this is the command that we use :

> ssh-copy-id user@IP

#### Disable the password authentification

We can secure way more our server by disable the password authentification, you gonna tell me that remove the password is not that secure **BUT** we gonna change this by a *passphrase* it's way more secure than a password cause if a hacker found its it will not know the passphrase so you server will be safe. To disable the password authentification we need to edit the **sshd_config** :

> sudo nano /etc/ssh/sshd_config
> 
>> PasswordAuthentification no

You have to found the PasswordAuthentification line then **uncomment** the line and type **no** instead of **yes**
Then you have to restard the SSH service to apply the change :

> sudo systemctl restart ssh

### Files transfer, SCP and SFTP

#### SCP

Now we have to see how to send files between machines, we gonna try with SCP. First we need to crate a file then we can use SCP: 

> echo "This is a test file" > file.txt
> 
> scp file.txt user@IP:/path/to/repositori

After you can check on the server if the file is well arrived.

#### SFTP

The only difference between SCP and SFTP is that SFTP have a error handling option that can help have a better view of the problem if the file transfer interrupt and if this is the case we can resume the transfer directly. 

> sftp user@IP
> 
> put file.txt

With the command above we can send a file from our local machine to the distant server, this command only work in a SFTP session.

### SSH Tunnels and Port Forwarding

#### Tunnels

An SSH tunnel is a secure way to connect to a remote server, allowing you to forward ports between your local machine and that server. In this case, you're setting up a tunnel to redirect local port 8080 to port 80 on a remote server.
Here's how it works: 
    
- Port Forwarding: This feature of SSH allows you to redirect network traffic from one port to another.

- Local Port Forwarding: When you set up local port forwarding, you can connect to a specific port on your local machine, and it will forward that traffic to a designated port on a remote server.

In your case:

- Local Port (8080): This is the port on your local machine where you will send requests.

- Remote Port (80): This is the port on the remote server that usually serves web traffic (HTTP).

Here is the command we use: 

> ssh -L 8080:localhost80 user@IP

Then you can go on the navigator to open the application on the local port : **http://localhost:8080**

#### Port Forwarding

The goal of this operation is to reduce brute-force automatic attack by changing the default port. We gonna put a firewall too for a more secure connection.
First of all we have to be connected to the SSH session then we need to make change in a file:

> sudo nano /etc/ssh/sshd_config
>
>> Port 2222

We found the line **Port** uncomment it then change the default port (22) to another port we want, for the exercise we chose 2222. After you have to restart the SSH server with the same command in the **Disable Password** part. To connect to the new port you have to specify it on the command : 

> ssh -p 2222 user@IP

#### Firewall

We gonna use UFW.
Configuring UFW to allow SSH connections only on a non-standard port like 2222 enhances your server's security by reducing exposure to automated attacks. It’s a simple but effective way to bolster your overall defense strategy.
I'm gonna list all the command you need to set up a UFW to secure your sever by only accept the entry of 2222 port, you need to install the firewall directly on the server : 

> sudo apt install ufw
>
> sudo ufw allow 2222/tcp
>
> sudo ufw deny 22/tcp
>
> sudo ufw enable
>
> sudo ufw status

Now if you try to connect without the **-p 2222** the connection is refused.

### Fail2Ban

In the same spirit as the forwarding port Fail2Ban is a tool to protect from brute force attack by blocking the IP who made to many try to connect.

> sudo apt install fail2ban
>
> sudo nano /etc/fail2ban/jail.local
>
>> [sshd]
>> 
>> enable = true
>> 
>> port = 2222
>>
> sudo systemctl restart fail2ban

With the above command you can set up fail2ban.

### Wireshark

With Wireshark we can view how all of this work. For those who don't now Wireshark is the most used packet analyzer. So open wireshark, on the application you have to choose the network interface you want to listen. Then you have to specify that you want to listen to the correct SSH session : **tcp.port == 2222** you can enter this in the filter on the top of the screen. Start the packet capture by clicking the green button, then we need to create trafic in our SSH session, so we connect to it then execute a few command. After doing this we can now analyze the result that appear on Wireshark, we can see the protocol of each packet, also the SYN and ACK and the SSH packet **but** the are crypted that's we we use SSH and not FTP (who display clear text).
We also can try fail2ban test to fail connect and see if the IP get blocked. The result will appear in Wireshark too.

## Problematic


