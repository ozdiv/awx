
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
