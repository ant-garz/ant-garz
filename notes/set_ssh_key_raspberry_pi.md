# How to set up an ssh key for a raspberry pi for remote access with a windows computer
These are notes that I have written as a reference and starting point when setting up a new raspberry pi for remote development.
A good resource I used for learning how to set up these keys is this youtube video ["Linux/Mac Tutorial: SSH Key-Based Authentication - How to SSH Without a Password"](https://youtu.be/vpk_1gldOAE?si=Tpzh-POLWWQs0JKL)

I find it easier to write programs on the raspberry pi with a remote connection set up in VSCode on my windows computer.
However, if you don't set up an ssh key and use password based authentication, the VSCode client often will continuously ask you for your password.

## Prerequisites
### Enable and configure SSH on your host (windows)
Detailed steps on how to enable SSH on your computer [Enable SSH on Windows](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=gui#install-openssh-for-windows)

### Enable SSH on your raspberry pi
Detailed steps to enable ssh on your raspberry pi can be found here [Enable SSH on the Raspberry Pi](https://raspberrypi-guide.github.io/networking/connecting-via-ssh#enable-ssh-on-the-raspberry-pi)

### Find local IP address of your raspberry pi
To find the raspberry pi's IP address, open a terminal and enter in `hostname -I`. Note your address for later.

## Step 1: Start key generation
On your windows machine, open a terminal and enter in `ssh-keygen -t rsa -b 4096`
More information on what this command is doing can be found here [Ssh-keygen](https://en.wikipedia.org/wiki/Ssh-keygen)
the `-t` argument indicates the type of key to create, in this case we are using an rsa key type.
the `b` argument indicates the number of bits we want to use for the key, in this case 4096 over the default value for RSA of 3072.

## Step 2: Decide where to save the key (as prompted from output of step 1)
An example prompt of what the first step will yield can be found below
```
PS C:\Users\username> ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\username/.ssh/id_rsa):
```
Since I like to use multiple machines, I organize my keys into subfolders of the `/.ssh` directory.
__Please note__: You'll want to ensure the sub-folder you want to save to exists before entering the filepath, otherwise the command will error out on you.
An example subfolder would look something like `C:\Users\username/.ssh/sub_folder/id_rsa`

## Step 3 Copy the public key from your windows machine to your remote machine
To copy a file from windows to linux (raspberry pi) we can use the `scp` command. From the Windows OpenSSH documentation linked above, we know that `scp` "is a file copy utility that runs on SSH".

__Please note__: You'll want to ensure the `/.ssh` folder exists on your remote machine before trying to copy over into it.

The command syntax looks like 
`scp "C:\Users\username\.ssh\subfolder\id_rsa.pub" remote-machine-name@remote-ip-address:/home/username/.ssh/sshkey.pub`

## Step 4: Append contents of pub key file on remote machine to Authorized Keys
In your raspberry pi, open a terminal after successfully copying the key over and enter in the following command
`cat ~/.ssh/sshkey.pub >> ~/.ssh/authorized_keys`

## Step 5: Set permissions for keys
On your raspberry pi, you will want to make the following permission updates
### SSH folder
Permissions should be set to 700 for this folder. Doing this will give read, write, and execute permissions for the user only.
The command to make this update is `chmod 700 ~/.ssh/`

### SSH folder contents
Permissions should be set to 600 for the contents of this folder. Doing this, you can read and write the file or directory and other users have no access to it.
The command to make this update is `chmod 600 ~/.ssh/*`

## Step 6 Turn off password authentication on remote machine
On your raspberry pi, make back up of your initial sshd config before making this update for safety. 
You can make a backup of that file by using the following command
`sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak`

Open your sshd_config file in your favorite text editor (I choose nano) and find the following lines
```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes
```
Change `PasswordAuthentication yes` to `PasswordAuthentication no`
Save your changes and exit the file.

After making this changes, restart the ssh server to reflect changes with 
`sudo service ssh restart`

## Step 8: Configure SSH client on host (windows) machine to include new client
On your windows machine there will also be a `/.ssh` folder containing your keys for use. In this folder open the config file in a text editor. 
Here, we need to add ssh clients in the following manner

```
Host SERVERNAME
Hostname ip-or-domain-of-server
User USERNAME
PubKeyAuthentication yes
IdentityFile ./path/to/key
```
Fill out this information with your applicable data and save.

## Wrap up
After these changes are made, you should be able to connect to your raspberry pi from your windows computer via passwordless SSH. Congrats!