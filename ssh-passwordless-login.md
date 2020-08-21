# SSH and Passwordless login

## Generating keypairs

Move into the ~/.ssh/ directory in your user home.

```
cd ~/.ssh/
```

Then generate a new keypair through the ssh-keygen command.

```
ssh-keygen -t rsa -b 4096 -C "EMAIL-OR-NICKNAME"
```

You will be prompted to choose a KEYNAME and a passphrase for your keypair.

This command will create two new keys into the current directory.

- (*private key*) KEYNAME
- (*public key*) KEYNAME.pub

## Copying the public key to the server

To copy your public key inside the USERNAME's ~/.ssh/authorized_keys file.

```
ssh-copy-id -i ~/.ssh/KEYNAME.pub USERNAME@IP-ADDRESS
```

## Create an SSH alias

The usual command to open an ssh connection looks like the following.

```
ssh -i ~/.ssh/KEYNAME.pub USERNAME@IP-ADDRESS
```

To connect via ssh using an alias instead of the usual verbose syntax alias for connections can be defined in a "config" file inside the user ~/.ssh/ directory.

---

*config*

```
Host server-alias-name
    Hostname IP-ADDRESS
    User USERNAME
    IdentityFile ~/.ssh/KEYNAME
    Port 22
```

---

Now it's possible to establish a connection with just:

```
ssh server-alias-name
```

## Disable Login with password

Once logged into the server open and edit the /etc/ssh/sshd_config file. Find the following lines and change them.

---

*sshd_config*

```
...
PasswordAuthentication no
ChallengeResponseAuthentication no
...
UsePAM no
...
```

---

To make changes effective restart the ssh service.

```
sudo systemctl restart ssh
```

Enjoy!
