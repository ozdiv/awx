# Installation of AWX (Opensource Ansible Tower) on RHEL 7 or 8
## Prerequisites
- At least 8GB of RAM, 16GB is better
- 4 vcpus minimum
- 30GB disk space minimum

## Step 1: Initial preparation
### Update your system
```sudo dnf -y update```

### Disable Firewalld. This is recommended by K3s.
```sudo systemctl disable firewalld --now```

### Disable nm-cloud-setup. This is recommended by Rancher.
```systemctl disable nm-cloud-setup.service nm-cloud-setup.timer```

```reboot```

## Step 2: Install K3s Kubernetes Distribution
Latest AWX can only run on Kubernetes. k3s.io is light version of kubernetes.

### Put SELinux in permissive mode:

```sudo setenforce 0```

```sudo sed -i 's/^SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config```

```cat /etc/selinux/config | grep SELINUX=```

### Install k3s by running the commands below:
Install container-selinux and selinux-policy-base

```yum install -y container-selinux selinux-policy-base```

```yum install -y https://rpm.rancher.io/k3s/stable/common/centos/8/noarch/k3s-selinux-0.3-0.el8.noarch.rpm```

```curl -sfL https://get.k3s.io | sudo bash -```

### As root user do a validation on use of kubectl Kubernetes management tool:
```
sudo su -

kubectl get nodes
```

You should see:
```
NAME     STATUS   ROLES                  AGE   VERSION
centos.hirebestengineers.com   Ready    control-plane,master   111s   v1.21.2+k3s1

```
You can also confirm Kubernetes version deployed using the following command:
```
kubectl version --short
```
You should see:
```
Client Version: v1.21.3+k3s1
Server Version: v1.21.3+k3s1
```


Step 3: Deploy AWX Operator on Kubernetes
-----------------------------------------

This Kubernetes Operator has to be deployed in your Kubernetes cluster, which in our case is powered by K3s. The operator we’ll deploy can manage one or more AWX instances in any namespace.

Check all currently running pods in all namespaces:

    $ kubectl get pods -A
    NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
    kube-system   metrics-server-86cbb8457f-8kk5w           1/1     Running     0          5m4s
    kube-system   local-path-provisioner-5ff76fc89d-jwmc7   1/1     Running     0          5m4s
    kube-system   coredns-7448499f4d-bk4r7                  1/1     Running     0          5m4s
    kube-system   helm-install-traefik-crd-mm782            0/1     Completed   0          5m4s
    kube-system   helm-install-traefik-zbwwl                0/1     Completed   1          5m4s
    kube-system   svclb-traefik-k7kjh                       2/2     Running     0          4m52s
    kube-system   traefik-97b44b794-hdlfg                   1/1     Running     0          4m52s

Run the command below to deploy AWX Operator into your cluster:

    kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/b684a5de35fb6cd57719a6d604ca0032cf44acf2/deploy/awx-operator.yaml

Command output:

    customresourcedefinition.apiextensions.k8s.io/awxs.awx.ansible.com created
    customresourcedefinition.apiextensions.k8s.io/awxbackups.awx.ansible.com created
    customresourcedefinition.apiextensions.k8s.io/awxrestores.awx.ansible.com created
    clusterrole.rbac.authorization.k8s.io/awx-operator created
    clusterrolebinding.rbac.authorization.k8s.io/awx-operator created
    serviceaccount/awx-operator created
    deployment.apps/awx-operator created

Wait a few minutes and _awx-operator_ should be running:

    $ kubectl get pods
    NAME                            READY   STATUS    RESTARTS   AGE
    awx-operator-545497f7d5-dgjs8   1/1     Running   0          84s

Step 4: Install Ansible AWX
---------------------------

Create Static data PVC:

    cat <<EOF | kubectl create -f -
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: static-data-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      storageClassName: local-path
      resources:
        requests:
          storage: 1Gi
    EOF

Let’s now create AWX deployment file with basic information about what is installed:

    vim awx-deploy.yml

Paste below contents into the file:

    ---
    apiVersion: awx.ansible.com/v1beta1
    kind: AWX
    metadata:
      name: awx
    spec:
      service_type: nodeport
      projects_persistence: true
      projects_storage_access_mode: ReadWriteOnce
      web_extra_volume_mounts: |
        - name: static-data
          mountPath: /var/lib/awx/public
      extra_volumes: |
        - name: static-data
          persistentVolumeClaim:
            claimName: static-data-pvc

We have defined resource name as **awx** and service type as **nodeport** to enable us access AWX from the Node IP address and given port. We also added extra PV mount on the web server pod.

Apply configuration manifest file:

    $ kubectl apply -f awx-deploy.yml
    awx.awx.ansible.com/awx created

Wait a few minutes then check AWX instance deployed:

    $ kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator"
    NAME                   READY   STATUS    RESTARTS   AGE
    awx-postgres-0         1/1     Running   0          75s
    awx-7c5d846c88-mjlvm   4/4     Running   0          64s

If you experience any issues with the Pods starting check deployment logs:

    $ kubectl logs -f deployments/awx-operator

The database data will be persistent as they are stored in a persistent volume:

    $ kubectl get pvc
    NAME                      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    postgres-awx-postgres-0   Bound    pvc-c0149545-8631-4aa1-a03f-2134a7f42aa6   8Gi        RWO            local-path     84s
    static-data-pvc           Bound    pvc-6b6005de-0888-4634-b0a2-d5cc92eb85cc   1Gi        RWO            local-path     2m10s
    awx-projects-claim        Bound    pvc-91e751e9-0e8e-40c8-9953-f8d9db5f612b   8Gi        RWO            local-path     77s

Volumes are created using local-path-provisioner and host path

    $ ls /var/lib/rancher/k3s/storage/
    pvc-edb29795-7dae-4a00-805f-2d989694fe3d_default_postgres-awx-postgres-0

Step 5: Access Ansible AWX Dashboard
------------------------------------

List all available services and check awx-service Nodeport

    $ kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
    NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    awx-postgres   ClusterIP   None           <none>        5432/TCP       2m5s
    awx-service    NodePort    10.43.182.53   <none>        80:32708/TCP   116s

You can edit the Node Port and set to figure of your preference, but has to be >=**32000**

    $ kubectl edit svc awx-service
    ....
    ports:
      - name: http
        nodePort: 32000
        port: 80
        protocol: TCP
        targetPort: 8052

Confirm if the change was updated successfully:

    $ kubectl get svc -l "app.kubernetes.io/managed-by=awx-operator"
    NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    awx-postgres   ClusterIP   None           <none>        5432/TCP       8m58s
    awx-service    NodePort    10.43.182.53   <none>        80:32000/TCP   8m49s

Ansible AWX web portal is now accessible on **http://hostip_or_hostname:32000**.

You are presented with the welcomedashboard.

The login username is **admin**

Obtain admin user **password** by decoding the secret with the password value:

    kubectl get secret awx-admin-password -o jsonpath="{.data.password}" | base64 --decode

Better output format:

    kubectl get secret awx-admin-password -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'

Login with admin username and decoded password.
