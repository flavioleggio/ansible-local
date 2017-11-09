# ansible-local

There are some requirements to execute the base playbook on a new host.
First, the playbooks in this repository assume that you have configured your hosts to have two network interfaces:

* NAT
* Host Only (Network 192.168.99.x)

After having created the virtual host (for example in Virtualbox) you have to ensure that `sudo` and `python-minimal`
are installed on it, so go for

```bash
su
apt-get update
apt-get install sudo
apt-get install python-minimal
```

If the user is not sudo-capable, you also have to add it to the sudoers.
In Debian you can simply add in the `/etc/sudoers.d/` directory a file like this

```
virtual_host_user   ALL=(ALL) ALL
```

Moreover, Ansible needs ssh access to the virtual host, so grant access to the master host adding its key to the virtual host
authorized keys. One way to simplify the operations is to create a NAT rule in Virtualbox for the virtual host, forwarding TCP
traffic from the address 127.0.0.1 on port 2222 to the virtual host on port 22. In this way you can add your key with this
command (on the master host)

```bash
ssh-copy-id virtual_host_user@virtual_host -p 2222
```

Now that the virtual host is Ansible-ready, you can add a new entry in the inventory file, containing the information that
Ansible uses to get access, for example

```
[host_group]
hostname ansible_host=127.0.0.1 ansible_port=2222
```

To play the base playbook, cd into the repository directory and run the following command
```
ansible-playbook -i inventory -u virtual_host_user -l hostname -sK base.yml
```

After playing the base playbook, you can change the entry with the actual host ip configured in
`roles/base/files/hosts/hostname/etc/network/interfaces`, or directly with the hostname if you have it in your master
host `/etc/hosts` file.
