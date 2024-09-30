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
    >PasswordAuthentification no

You have to found the PasswordAuthentification line **uncomment** the line and type **no** instead of **yes**
Then you have to restard the SSH service to apply the change :

    > sudo systemctl restart ssh
