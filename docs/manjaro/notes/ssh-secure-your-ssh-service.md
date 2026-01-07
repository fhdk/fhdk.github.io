---
title: 'SSH server security'
taxonomy:
    category:
        - docs
---

**Difficulty: ★★★☆☆**

This document has been posted at [Manjaro Forum][10] which in turn is put together from my old topics from the archived forum [[1][11]] [[2][12]]

## SSH server security
When you setup a SSH server on a public IP, your service will be ***spammed within minutes*** attempting to brute force your login.

Your first task as admin of a SSH server is to secure it.

## Topic assumptions

1. A Manjaro instance with at least two users, a root user __and__ a user belonging to the ` %wheel ` group.
2. A drop-in config in `/etc/sudoers.d/` assigning permissions to the group.
3. Such file is created with default Manjaro install
   ```
   $ cat /etc/sudoers.d/10-installer
   %wheel ALL=(ALL) ALL
   ```

If you have installed Manjaro by hand you will need to establish such file by hand. **Always use visudo** as the filepermissions are important.

## SSH security
Your SSH server is not the only server you may come across using a public key infrastucture. If you are a using services like Github, GitLab, SourceForge, OSDN and many more - you have come across this before.

## 1. Public key login

### Create key pair
SSH public key infrastructure is actually a key *pair* consisting of both a public and a private key. The public part has the extension **.pub** and this is the key you place on the server or service.

You create key pair using the `ssh-keygen` utility. This utility takes various arguments which affects how the key-pair is generated and where the files are placed.

The defaults are sane but in case you want a stronger key, you can use the man-pages to expand your knowledge.

```bash
$ man ssh-keygen
```
It is recommended to use a separate key for each server and each device you are using to connect with and to tell the keys apart you use a different filename  e.g. the name of the service. If you have a [Linode][2] cloud instance you could name the file **linode** or if you are using [OSDN][3] you may name it **osdn**.

To create a key-pair for a service using the service name for the key-pair, you supply the path to the file using the **-f** argument - in this example I will use the name __ssh-service__ with the cipher appened.

When you create the key you have the option of using a password - or you can leave password blank. This of course lowers the security on client side - but you don't have to unlock the key-pair on use - and it is a matter of choice - just remember to keep you private keys safe - no matter using key password or not.

__EDIT: 2021-10-16__
Due to recent changes and deprecation of ciphers considered unsafe you should add the `-t` option to specify the cipher to be used preferably __ed25519__

```bash
$ ssh-keygen -t ed25519  -f ~/.ssh/ssh-service-ed25519.ppk
```

You don't need to use the `.ppk` part, but doing so makes it easy to configure [FileZilla][4] as SFTP client using a keyfile.

### Transfer the public key to the server
Many VPS providers only provide a root account on the initial device in which case you will have to target root for the keyfile.

Manjaro installations defaults to keyfile only for root login. Therefore you will need another user account on the device with sudo privilege to be able to upoload the keyfile.

To upload the public part to the server - you can use the `scp` command which is an abbreviation of **secure copy**. The command consist of three parts:

* The command itself
* The path to the file to be copied 
* Server and where to put the file

In case of a VPS with only a root user
```
$ scp ./ssh/ssh-service-ed25519.ppk.pub  root@host:/root/.ssh
```
Or in case of a rapsberry pi in your local network
```
$ scp ./ssh/ssh-service-ed25519.ppk.pub  $user@host:/home/$USER/.ssh/
```

Another option is to use [ssh-copy][1] to send the file to your ssh service
```bash
ssh-copy-id -i ~/.ssh/ssh-service-ed25519.ppk.pub $USER@server.domain.tld
```

Log off the server and test your connection using your identity file
```bash
ssh $USER@server.domain.tld -i ~/.ssh/ssh-service-ed25519.ppk.pub
```
If you - in the above key-generation supplied a password to protect your key-pair - you will be prompted to unlock the key-pair.

If you did it right - you are now in a  remote shell without being prompted for credentials. 

## 2. Disable password login 
When you are logged in - using the public key - edit the file `/etc/ssh/sshd_config`

```bash
sudo nano /etc/ssh/sshd_config
```
Scroll down to the lines reading:
```text
# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no
```
And change them  to:
```text
PasswordAuthentication no
```

## 3. Change port
While you are logged in - **seriously consider changing the port from the default 22 to something else!** Consult `/etc/services` to avoid a collision with known services - but in reality any port over 10000 can be used.

The port is located at the top of the configuration
```text
# Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```

Uncomment the line  and change the port number
```text
Port 33000
```

Save the changes and restart the sshd daemon
```bash
# systemctl reload sshd
```

## 4. Connecting to the server
Connecting to SSH server can be automated by means of the user's local configuration.  Some points to remember:
- the file is parsed from top to bottom  
- the first match is used 

*so don't keep duplicated entries!*

If you plan to connect to the same server using different users, this can easily be done using different names for the **Host** entry.

Every server or service is designated by the **Host** - all lines between **Host** entries belong to the preceding **Host**.

```bash
$ nano ~/.ssh/config
```
Add your server details - examples
```text
Host nickname
  Hostname server.domain.tld
  IdentityFile ~/.ssh/ssh-service-ed25519.ppk
  user root
  Port 33000
Host fido.domain.tld
  IdentityFile ~/.ssh/fido.ppk
  user fido
```
```bash
ssh nickname
```

### Note on host key fingerprint
Upon first connection to a - previously unknown - ssh service you will be challenged before connection - something like

```
$ ssh user@hostname
The authenticity of host 'hostname (v.x.y.z)' can't be established.
ED25519 key fingerprint is SHA256:P4QBIqLt6g6JU5P3po0WRLF+mr0ypYhhG3iGgCprM20.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:15: v.x.y.z
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

The ssh client will always try to match the remote system to a list of known hosts found in ~/.ssh/known_hosts and if no match is found the client challenge you to trust the host. If you accept the challenge - the host key fingerprint will be added to the list.

If you - for any reason want to skip this check you can do so by adding an option to your connection command

    ssh -o StrictHostKeyChecking=no

If you are wearing a blackhat and have a host which regularly changes IP or if you only connect to your own systems you could opt to completely ignore the fingerprints by adding it to your local __~/.ssh/config__

```
Host *
  StrictHostKeyChecking no
```

### A note on passwordless keyfiles

When you have a lot of key-pairs (using no password) - you may run into the message - **too many failed logins** and subsequently - on a Manjaro system - the user may be locked.

You can create a workaround for this by disabling identity agent for the host or you can specify the the identity file on the command line when you connect to the host.

This can be done for all or for a single host
```
Host nickname
  IdentityAgent none
...
```

Or using a wildcard
```
Host *
  IdentityAgent none
...
```

## 4. Secure your key-pairs
You don't want to loose your key collection.

**IMPORTANT**: Keep a copy of your **~/.ssh** folder in a secure location and if you are using password less keys - keep them safe by:

- using a strong password on your system 
- disabling the root account
- mandatory `chown $USER:$USER .ssh -R && chmod u-x,go-rwx ~/.ssh/*`

## 5. Firewall
Next step is to ensure **only** allowed services are accessible by implementing a firewall.

### Firewalld
[Firewalld][5] a very good firewall service - well suited on systemd based systems like Manjaro. Firewalld can be configured using the term application since an application is merely a definition of which ports should be allowed - e.g. a http application or ssh or smtp.

When you configure the firewall you use zones to define where you are and services to define what you allow. Install firewalld:

```bash
# pacman -Syu firewalld
```

When firewalld is enabled and started the default zone is **public** which allows the computer to be visible but all ports closed.

Adding a specific service (application) is easily done using the command line but you can also use the applet found in your system configuration area. A systray application is available if you install the dependencies for it (python-pyqt5). Adding services has immediate effect - no need to reload the service. Simply add the service to the allowed service to the desired zone

Example - adding **http** to **public** zone
```bash
# firewall-cmd --zone=public --add-service=http
success
```
It is important to realize that changes you make on the fly are not permanent. To make a certain service available permanently, add the `--permanent` argument

```bash
# firewall-cmd --permanent --zone=public --add-service=http
success
```
What if you want to add your own service definition?

Easy-peasy - look in the folder `/usr/lib/firewalld/services` and make a copy of an appropriate service definition.

Example - you want to run a ssh server on a non default port.

Copy the `ssh.xml` service definition to `/etc/firewalld/services`
```
# cp /usr/lib/firewalld/services/ssh.xml /etc/firewalld/services/my-ssh.xml
```
Edit the service definition 
```bash
# nano /etc/firewalld/services/my-ssh.xml
```
Change the port to match your service and the short name to distinguish from the original service.
```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>My SSH service</short>
  <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
  <port protocol="tcp" port="30000"/>
</service>
```
Wait 5-10 seconds for the service file to be recognized and activate it (Same rule on permanent applies)
```bash
# firewall-cmd --zone=public --add-service=my-ssh --permanent
success
```
That's it - more info on firewalld can be found using the dedicated webpage

https://firewalld.org/documentation/

[1]: https://www.ssh.com/ssh/copy-id/#copy-the-key-to-a-server
[2]: https://linode.com
[3]: https://osdn.net
[4]: https://filezilla-project.org
[5]: https://firewalld.org/
[10]: https://forum.manjaro.org/t/root-tip-set-up-your-own-ssh-service/60665
[11]: https://archived.forum.manjaro.org/t/howto-secure-ssh-server-connect-using-ssh-keys/117639
[12]: https://archived.forum.manjaro.org/t/howto-secure-your-device-using-firewalld/117665