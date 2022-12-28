# Kompose, translate a Docker Compose File to Kubernetes Resources



###### By Juan Manuel Payán / jpaybar

st4rt.fr0m.scr4tch@gmail.com



`Docker-compose` is an excellent tool for defining and running docker-based multi-container applications. But, what happens if you want to migrate to using `Kubernetes` and you don't know or are just starting out?

This is where `Kompose` can make it easier for you to get started on `Kubernetes`. 

**In this tutorial we will explain how to install Kompose and how to convert a basic docker-compose file that deploys WordPress into a Kubernetes manifest.**



### Previous requirements

In order to follow the tutorial, you must have `minikube` installed or some other method to access a Kubernetes cluster and `kubectl`.

### What's Kompose?

`Kompose` is a tool that allows you to:

- One command to build an application on `Kubernetes` using the compose file definition.

- Generate the `manifest` files necessary to launch the application.



https://kompose.io/



### Installation

We have multiple ways to install `Kompose`. Preferred method is downloading the binary from the latest GitHub release and add the binary to your PATH.

[Kompose - Installation](https://kompose.io/installation/)



**Linux and macOS:**

```
# Linux
curl -L https://github.com/kubernetes/kompose/releases/download/v1.27.0/kompose-linux-amd64 -o kompose

# macOS
curl -L https://github.com/kubernetes/kompose/releases/download/v1.27.0/kompose-darwin-amd64 -o kompose

chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose
```

**Windows:**

Download from [GitHub](https://github.com/kubernetes/kompose/releases/download/v1.27.0/kompose-windows-amd64.exe) and add the binary to your PATH. (First method)

```powershell
Invoke-WebRequest -OutFile 'kompose' -Uri 'https://github.com/kubernetes/kompose/releases/download/v1.27.0/kompose-windows-amd64.exe' -UseBasicParsing
```

```powershell
cp .\kompose C:\Windows\System32\
```

`Kompose` can be installed via [Chocolatey](https://chocolatey.org/packages/kubernetes-kompose) as well (Second method)

```
choco install kubernetes-kompose
```

Previously you need to have `Chocolatey` installed on your system.

1.- Start and type `powershell`. 

2.- We right click on Windows `Powershell` and choose `Run as administrator`. 

3.- We `paste the following command in Powershell` and press enter.

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; `
 iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

4.- We answer Yes when requested. 

5.- We close and reopen an elevated PowerShell window to start using Chocolately or just run RefreshEnv.cmd.



### The docker-compose file which deploys WordPress:

```yaml
version: "3.8"

services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: user
      WORDPRESS_DB_PASSWORD: password
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html
  mysql:
    image: mysql:latest
    restart: always
    ports:
      - 3306
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_RANDOM_ROOT_PASSWORD: "1"
    volumes:
      - mysql:/var/lib/mysql

volumes:
  wordpress:
  mysql:

```



### Generate Kubernetes files

We will use a docker-compose file to generate the YAML needed to get the application up on Kubernetes. Later, we can edit the files before launching the application. Also, it provides a good starting point to better understand how to write these files.

To do this, you must run the command:

```powershell
kompose convert -f docker-compose.yml
```

And we will see an output similar to the following:

```powershell
INFO Kubernetes file "mysql-service.yaml" created
INFO Kubernetes file "wordpress-service.yaml" created
INFO Kubernetes file "mysql-deployment.yaml" created
INFO Kubernetes file "mysql-persistentvolumeclaim.yaml" created
INFO Kubernetes file "wordpress-deployment.yaml" created
INFO Kubernetes file "wordpress-persistentvolumeclaim.yaml" created
```

As we can see, 3 files have been created for each service (WordPress and MySQL).



### Manifest files

A `Kubernetes manifest` file includes instructions in a `yaml` or `json` file that specify `how to deploy` an application to the node(s) in a `Kubernetes cluster`. The instructions include information about the `Kubernetes deployment`, the `Kubernetes service`, and other Kubernetes objects that will be created in the cluster. The manifest is also often referred to as a pod specification or deployment.yaml file (although other file names are allowed).

Our `Kompose` tool has generated 3 files for each service found in the `docker-compose.yaml` file. Let's take a look at one of them, for example at the `wordpress-service.yaml` file:

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: C:\ProgramData\chocolatey\lib\kubernetes-kompose\tools\kompose.exe convert -f docker-compose.yml
    kompose.version: 1.27.0 (b0ed6a2c9)
  creationTimestamp: null
  labels:
    io.kompose.service: wordpress
  name: wordpress
spec:
  ports:
    - name: "80"
      port: 80
      targetPort: 80
  selector:
    io.kompose.service: wordpress
status:
  loadBalancer: {}

```

The only drawback of generating the files with `Kompose` is the large number of specific tags that the tool leaves in the file. It is up to the preference of each one whether to delete them or not.

```yaml
annotations:
    kompose.cmd: C:\ProgramData\chocolatey\lib\kubernetes-kompose\tools\kompose.exe convert -f docker-compose.yml
    kompose.version: 1.27.0 (b0ed6a2c9)
  creationTimestamp: null
```

As we can see, it has created a service type file in addition to two other files (`wordpress-deployment.yaml` and `wordpress-persistentvolumeclaim.yaml`). One for the deployment itself and another for the necessary volumes.
The same files have also been created for the `MySQL service` described in the `docker-compose.yaml` file.



### Create the resources in Kubernetes

Then you can use `kubectl apply` to create these resources in kubernetes.

```powershell
kubectl apply -f .
```

To view created resources:

```powershell
kubectl get all
```



### Access the WordPress service

To check that our `WordPress` service is working correctly and for testing since this is not the correct way to access a `Kubernetes` resource, we can execute the following command:

```powershell
kubectl port-forward service/wordpress 80:80
```

Now we can access WordPress from our browser:

![install_wordpress.PNG](https://github.com/jpaybar/Kubernetes/blob/main/kompose_docker-compose_WordPress/_images/install_wordpress.PNG)



As we have said before, this is not the correct way to access a service in `Kubernetes`. To access it correctly, we will make a modification in the `wordpress-service.yaml` file that `Kompose` has generated for us from our original `docker-compose.yaml` file.

##### There are 2 types of ports in Kubernetes:

`ClusterIP` and `NodePort` The first one is created by default for any service and allows access to it from within the Cluster, while the second allows us to access the resource from outside of Kubernetes.

Our new `wordpress-service.yaml` file would look like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: C:\ProgramData\chocolatey\lib\kubernetes-kompose\tools\kompose.exe convert -f docker-compose.yml
    kompose.version: 1.27.0 (b0ed6a2c9)
  creationTimestamp: null
  labels:
    io.kompose.service: wordpress
  name: wordpress
spec:
  type: NodePort
  ports:
    - name: "80"
      port: 80
      targetPort: 80
  selector:
    io.kompose.service: wordpress
status:
  loadBalancer: {}

```

We have added the following tag:

```yaml
type: NodePort
```

Now, if we run this command:

```powershell
kubectl get svc
```

We will see a similar output:

```powershell
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        4d16h
mysql        ClusterIP   10.96.40.171    <none>        3306/TCP       22m
wordpress    NodePort    10.109.56.252   <none>        80:31110/TCP   22m
```

As we can see, our `WordPress` service is now of the `NodePort` type.

If we’re using `minikube` we may access it via the `minikube service` command.

```powershell
minikube service wordpress
```

```powershell
|-----------|-----------|-------------|-----------------------------|
| NAMESPACE |   NAME    | TARGET PORT |             URL             |
|-----------|-----------|-------------|-----------------------------|
| default   | wordpress | 80/80       | http://192.168.59.120:31110 |
|-----------|-----------|-------------|-----------------------------|
```

Now we can access `WordPress` from our browser. The URL will be the IP of the node of our cluster and the port that `Kubernetes` has randomly generated and that will be between 30.000-40.000:

![wordpress_service.PNG](https://github.com/jpaybar/Kubernetes/blob/main/kompose_docker-compose_WordPress/_images/wordpress_service.PNG)



##### **NOTE:**

If we are running Minikube from VirtualBox, we must add a NAT rule like the following:

![nat_rule.PNG](https://github.com/jpaybar/Kubernetes/blob/main/kompose_docker-compose_WordPress/_images/nat_rule.PNG)





## Author Information

Juan Manuel Payán Barea    (IT Technician) [st4rt.fr0m.scr4tch@gmail.com](mailto:st4rt.fr0m.scr4tch@gmail.com)

[jpaybar (Juan M. Payán Barea) · GitHub](https://github.com/jpaybar)

https://es.linkedin.com/in/juanmanuelpayan
