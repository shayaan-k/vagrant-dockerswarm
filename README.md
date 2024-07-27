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

## Docker Setup
Docker is already instlled on nodes from ansible.

Create a simple flask app to run as a containterized application

Create a Dockerfile that specifies the enviroment the app should be run in.
I want to run this app on a Python environment so I specify a python linux environment for the base image.
I then want to specify the flask app i want to run as well as the IP. (0.0.0.0)
I can copy and then install the requirements file which containes the packages needed for my app to be runnable.
I expose port 5000 as the port to communicate with.
I copy the app to the working directory and run the ```flask run``` command to start the app

I have also created a docker compose file as I want my application to run on multiple nodes
This docker compose file simply exposes port 5000 on all nodes it runs on so my application can be run seemlessly in parrallel.

I can run the command ```docker-compose up``` to start my application on the node and test it by going into another node and running ```curl node1:5000```. This command should output Hello World as specified in the flask application

## Docker Swarm
I started by cloning another playbook that sets up the docker swarm enviroment for me.
For this portion I use node1 as the mastern and node 2 and 3 as workers.

After setting up my nodes, I can go into a node and write ```docker node ls``` and see all the nodes listed in the cluster. With Node 1 as the leader.

I now want to end the docker session that was running with ```docker-compose down```

Docker swarm does not build images but instead grabs them from image repositries. That means my docker compose file (which specifies to build the app) wont work. I instead need to grab a completed image and use that. 

I can buid and push an image like this:
```docker build -t my-flask-app .```
 > build image name my-flask-app using the Dockerfile in the current directory.

Tag the docker image to a remote location: 
```docker tag my-flask-app shsyaank/my-flask-app:latest```
 > shayaank/my-flask-app is the location it will be stored

Push the docker image:
```docker push yourusername/my-flask-app:latest```

Now I can modify the Docker Compose file to use the image.

To Start the application I run:
```docker stack deploy --compose-file docker-compose.yml myapp```

```docker stack ls``` provies basic info of running services
```docker stach services myapp``` shows more information about the running service

I can scale up the number of running services by running:
```docker service scale myapp_web=6```
```docker service ps myapp_web```

## Testing app

My going to the Control Node and curling and of the nodes I will get a 'Hello World I am xxx' response that I am expecting.