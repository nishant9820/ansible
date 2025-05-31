# How to setup Passwordless Authentication

---


# SSH Key-Based Authentication Setup Guide
---

## 1. Generate SSH Key Pair on Client (`node2`)

Run the following command on your client machine (`node2`):

```bash
ssh-keygen -t rsa -b 4096 -C "vagrant@node2"
````

* Press **Enter** to accept the default file location (`/home/vagrant/.ssh/id_rsa`)
* Optionally enter a passphrase or leave it empty for no passphrase
* This creates:

  * Private key: `/home/vagrant/.ssh/id_rsa`
  * Public key: `/home/vagrant/.ssh/id_rsa.pub`

---

## 2. Attempt to Copy Public Key to Remote Server

Use the built-in tool to copy the public key for passwordless login:

```bash
ssh-copy-id vagrant@192.168.56.12
```

### Possible Issue:

```plaintext
Permission denied (publickey).
```

---

## 3. Troubleshoot SSH Permission Denied Errors

To fix permission issues when `ssh-copy-id` fails, update the SSH server configuration on the **remote machine**.

### 3.1 Edit SSH daemon config

```bash
sudo vim /etc/ssh/sshd_config
```

Make sure these lines are set:

```plaintext
PubkeyAuthentication yes
PasswordAuthentication yes
```

> **Note:** `PasswordAuthentication yes` allows fallback to password login, which helps when your key is not yet authorized.

### 3.2 Restart SSH service to apply changes

```bash
sudo systemctl restart ssh
```

---

## 4. Manually Copy Public Key (If `ssh-copy-id` still fails)

### 4.1 On client (`node2`), display your public key:

```bash
cat ~/.ssh/id_rsa.pub
```

Copy the entire output.

### 4.2 On the remote server (`192.168.56.12`), prepare SSH directory:

```bash
mkdir -p ~/.ssh
vim ~/.ssh/authorized_keys
```

Paste the copied public key into the file, then save and exit.

### 4.3 Set correct permissions on remote:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## 5. Verify SSH Login

From the client (`node2`), test SSH login:

```bash
ssh vagrant@192.168.56.12
```

You should connect without being prompted for a password.

---

## 6. Optional: Improve Security by Disabling Password Login

After confirming key-based login works, disable password login on the remote server by setting in `/etc/ssh/sshd_config`:

```plaintext
PasswordAuthentication no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

---

## Summary

* Generated a 4096-bit RSA key pair.
* Copied public key to remote server.
* Configured remote SSH daemon to accept public keys and optionally passwords during setup.
* Manually added keys and fixed permissions on remote server if `ssh-copy-id` failed.
* Tested passwordless SSH login.
* Optionally disabled password authentication for security.

---

## Troubleshooting Tips

* Confirm user home and `.ssh` directory ownership and permissions.
* Confirm no duplicate keys or conflicting ssh-agent keys.
* Check `/var/log/auth.log` on remote for SSH errors.
* Verify the correct IP address and username.

---

## EC2 Instances

### Using Public Key

```
ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ubuntu@<INSTANCE-PUBLIC-IP>
```

- ssh-copy-id: This is the command used to copy your public key to a remote machine.
- -f: This flag forces the copying of keys, which can be useful if you have keys already set up and want to overwrite them.
- "-o IdentityFile <PATH TO PEM FILE>": This option specifies the identity file (private key) to use for the connection. The -o flag passes this option to the underlying ssh command.
- ubuntu@<INSTANCE-IP>: This is the username (ubuntu) and the IP address of the remote server you want to access.

### Using Password 

- Go to the file `/etc/ssh/sshd_config.d/60-cloudimg-settings.conf`
- Update `PasswordAuthentication yes`
- Restart SSH -> `sudo systemctl restart ssh`
