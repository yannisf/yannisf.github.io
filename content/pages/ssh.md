---
title:  "ssh"
date: 2023-08-15
---

## Key management

### Generate keys

```bash
mkdir ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t rsa
```

The private key should remain safe and secure on your client host. The public key should be moved to the `.ssh` directory of the remote host account you want to log in using key authentication.

**NOTE**: Keys should be generated on your secure machine and not on the machine you intend to install the public key on.

### Register key to remote host

```bash
cat id_rsa.pub >> authorized_keys
```

## Executing a command on the remote system

```bash
ssh host <command>
```

## Using advanced configuration

In order to invoke `ssh` with any of its advanced configuration options (these are typically set in `/etc/ssh/ssh_config` or in user configuration per host in `~/.ssh/config`) you can use the `-o` command syntax.

### Example

When there are any keys in your `.ssh` directory, ssh will try those keys against the server you are connecting. If there are many keys and none is appropriate you might get a `Too many authentication failures` error. Use the following command to only use password authentication:

```bash
ssh -o PreferredAuthentications=password user@host
```

## Port forwading

### Local port forwarding

```bash
ssh -L 8000:localhost:80 host <1>
ssh -L 8000:otherhost:80 host <2>
```

- <1> When connecting to `localhost:8000` you will get `host:80` as if connecting from `host`
- <2> When connecting to `localhost:8000` you will get `otherhost:80` as if connecting from `host`

**TIP**: It is worth noting, that local port forwarding, always opens a port in localhost and obviously you need ssh access to the host in order for the tunnel to be established.

### Remote port forwarding

```bash
ssh -R 8000:localhost:80 host <1>
ssh -R 8000:otherhost:80 host <2>
```

- <1> When connecting to `host:8000` you will get `localhost:80`
- <2> When connecting to `host:8000` you will get `otherhost:80` as if connecting from `localhost`

**NOTE**: The host your are connecting to (host) will need to have `GatewayPorts yes` in its `/etc/ssh/sshd_config`.

### Dynamic forwarding

```bash
ssh -D 1080 host
```

- Establishes a secure connection with host. In your proxy settings, define `localhost:1080` as a SOCKS proxy. Any network requests will be served through this proxy.

**TIP**: All port forwarding modes open a terminal on the target server. To avoid this, add the `-Nf` switches. To tear down the tunnel the specific ssh process must be found and killed.

## ssh-agent

Its role it twofold. First, it caches your keys enabling you to provide the passphrase just once. Second, it allows you to forward your private key to a second server (more on this later). If ssh-agent is not already running, there are two options to start it:

### Launching a new shell from within ssh-agent

```bash
ssh-agent bash
```

This will launch a new shell which the agent environment set.

#### Merging the appropriate environment variables to the current shell

```bash
eval `ssh-agent -s`
```

After that, keys should be added to the like so:

```bash
ssh-add ~/.ssh/id_rsa
```

**TIP**: `~/.ssh/id_rsa` is one of the default keys that `ssh-add` will look for, so it is not necessary to provide this information.

### ForwardAgent

Lets assume that you want to log in `host_c` from `host_a`. To log in `host_c` you have to use `host_b` as a jumphost. In `host_a`, private and public keys exist. and `host_b` and `host_c` are configured to accept your keys. This will require you first to log in `host_b` and then `host_c`. And this requires having your private key not just in `host_a`, but in `host_b` as well.

Using forwarding this can be avoided. Just login in `host_c` like so:

```bash
ssh -A host_b
ssh host_c
```

**NOTE**: ssh-agent must be running in host a for forwarding to work, and daemons must be configured appropriately.

**TIP**: The `-A` parameter can be skipped if `.ssh/config` is configured like so:

```config
Host host_b
  ForwardAgent yes
```

**WARNING**: Using `ForwardAgent` possibly opens a back door in the host you are loging in and if possible should be avoided. The `ProxyCommand` that will be presented in a following section is considered a safer option.

### ProxyCommand

Lets assume that you want to establish a connection from `host_a` to `host_c` via `host_b`. This functionality is better realized through configuration (`.ssh/config`). It assumes that the user key that has been installed in `host_a` is accepted in `host b` and `host_c`.

```config
Host c_via_b <1>
  HostName host_c <2>
  ProxyCommand ssh host_b -W %h:%p <3>
```

- <1> Configures c_via_b
- <2> Defines the target host
- <3> Uses `ProxyCommand` to reach hostC through hostB

Log in `host_c` via `host_b`

```bash
ssh c_via_b
```

**TIP**: Intermediate hosts can be added at will!

### ServerAliveInterval

When a connection is left idle, it is eligible to closure. This option sends every so many seconds a message to the server to keep the connection alive.

## sshfs

It is quite simple to mount a filesystem where you have ssh access.

### Install sshfs

```bash
sudo apt-get install sshfs
```

### Add user to fuse group

```bash
sudo gpasswd -a $USER fuse
```

**TIP**: For the new groups to take effect, the user has to log out and in again.

### Mount the remote filesystem

```bash
mkdir -p ~/mount_point <1>
sshfs server:/remote_directory ~/mount_point <2>
```

### Unmount

```bash
fusermount -u ~/far_projects
```

## Resources
- https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts
- https://developer.github.com/guides/using-ssh-agent-forwarding
- http://www.unixwiz.net/techtips/ssh-agent-forwarding.html
- https://heipei.github.io/2015/02/26/SSH-Agent-Forwarding-considered-harmful
- http://blog.trackets.com/2014/05/17/ssh-tunnel-local-and-remote-port-forwarding-explained-with-examples.html
