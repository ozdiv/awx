
Install docker.

```sudo yum install -y yum-utils```

```sudo yum-config-manager     --add-repo     https://download.docker.com/linux/centos/docker-ce.repo```

```sudo yum install docker-ce docker-ce-cli containerd.io```

```sudo systemctl enable docker```

Install portainer.

```sudo su```

```docker volume create portainer_data```

```docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce```

Once portainer is installed, navigate to http://10.0.19.237:9000/  or replace ip with your server, created user and log into portainer.

Click on local and find on the left-hand side App Templates. Click on 'App Templates' and click on 'GitLab Ce'. Click 'deploy the container'.

It takes a long time for GitLab container to be created, so be patient - takes 10-20 minutes at least.

After 20 minutes go to http://10.0.19.237:49154/ or replace with different ip of your server.

Default user is root. Initial password is located inside the container in this file /etc/gitlab/initial_root_password
