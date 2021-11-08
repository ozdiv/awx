### Install docker.

```sudo yum install -y yum-utils```

```sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo```

```sudo yum install docker-ce docker-ce-cli containerd.io```

```sudo systemctl enable docker```

### Install docker-compose

``` sudo curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose```
```sudo chmod +x /usr/local/bin/docker-compose```

Make an alias if needed

```alias docker-compose="/usr/local/bin/docker-compose"```

Only for RHEL7 if the error comes up - error while loading shared libraries...


```sudo mount /tmp -o remount,exec```

### Install Ansible

```sudo install -y ansible```

Check version

```ansible --version```

### Install node and npm

```curl -sL https://rpm.nodesource.com/setup_14.x | sudo bash -```
```sudo yum install -y nodejs npm```
```sudo npm install npm --global```

### Install and Setup Ansible AWX

```sudo apt install -y python3-pip git pwgen```

Check your docker-compose version first - ```docker-compose --version```

```sudo pip3 install docker-compose==1.29.0```

```sudo yum install unzip```

```wget https://github.com/ansible/awx/archive/17.1.0.zip```

Secret key is:

```Q0kpzxtu2I1LlP2oSUYhWIJqlDSzEo```


