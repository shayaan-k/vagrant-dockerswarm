# vagrant-dockerswarm
Project to provision servers with vagrant and run applications with docker swarm

# Project rundown
In this README, my goal is to document the entire process of completing this project. This will be my resource for working with these technologies in the future.

## Creating the Vagrantfile
The vagrant file is the setup script for provisioning the Virtual Machines
This github link is a really good resource for everything I need to know about Vagrant: https://gist.github.com/wpscholar/a49594e2e2b918f4d0c4

I started by creating an array called server which provides the basic information needed to provision the servers. It is better to use a loop to define each VM rather than hardcoding each VM individually.
The array includes a hostname (what the machine is called), an image (os that the machine is run on, bento OS is lightweight so its a good option for a small project), ip (defines ip to communicate to, all IPs are in the range of the Host-Only adapter), and ssh port (the ssh port is a portforward of port 22 so that there are no port conlficts)

I then loop through each item in 'server'. First I define the hostname of the VM for the Vagrant config. Then I define 'node' for each hostname. This 'node' holds VM specific information such as the info listen in 'server' array. Finally, I define 'vb' which hosts specific virtualbox configurations such as cpu and memory.

Once that is all done I can run ```vagrant up``` to spin up the machines.

## Ansible Setup
Install ansible on control node with ```sudo apt install ansible```

Create sshkey on control and copy it to other nodes. This is to allow ansible to create an ssh connection without prompting for a password

All ansible modules can be found on the Ansible documentation here: https://docs.ansible.com/ansible/latest/module_plugin_guide/modules_intro.html

Run ```ansible nodes -m ping``` to check connectivity.

run the ansible playbook to install docker and docker compose on the nodes.
