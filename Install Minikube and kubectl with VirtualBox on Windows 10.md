### What is Minikube?

Minikube is a tool that manages virtual machines where a cluster or rather a Kubernetes instance runs on a single node. However Minikube relies on a hypervisor, which by default is VirtualBox. So Minikube helps us to run Kubernetes on a single node on our machine. To do this VirtualBox starts a linux virtual machine from an iso image called `boot2docker.iso`.

The Boot2Docker distribution is based on Tiny Core Linux and runs completely from RAM. The ISO installation occupied 27 MB. Boot2Docker started up in around 5 seconds.

### Installing Minikube and kubectl on windows 10

[minikube start | minikube](https://minikube.sigs.k8s.io/docs/start/)

Run the following commands:

```bash
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
```

```bash
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
```

```bash
Invoke-WebRequest -OutFile 'c:\minikube\kubectl.exe' -Uri 'https://dl.k8s.io/release/v1.24.0/bin/windows/amd64/kubectl.exe' -UseBasicParsing
```

```bash
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){ `
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine) `
}
```

Open a new "cmd" window to refresh the new Path.

### Running Minikube

```bash
minikube start
```

If we are behind a proxy, we must open a "CMD" console and configure our proxy:

```bash
set http_proxy=http://proxy.xxxx.xxxx.net:8080
set https_proxy=http://proxy.xxxx.xxxx.net:8080
```

This way you will be able to download the image and boot the VM on VirtualBox, but you will not be able to configure the cluster during the installation and a red exclamation mark will appear saying that the IP address of the Node is not excluded from the Proxy, we will have to add it by executing:

```bash
set no_proxy=node_IP
```

Now, you may run:

```bash
kubectl get pods -A
```

If you want to log in to the node you can execute the following command:

```bash
minikube ssh
```

or

```bash
ssh docker@node_IP    (passwd "tcuser")
```

## Manage your cluster

Pause Kubernetes without impacting deployed applications:

```shell
minikube pause
```

Unpause a paused instance:

```shell
minikube unpause
```

Halt the cluster:

```shell
minikube stop
```

Increase the default memory limit (requires a restart):

```shell
minikube config set memory 16384
```

Browse the catalog of easily installed Kubernetes services:

```shell
minikube addons list
```

Activate an add-on:

```bash
minikube addons enable "myaddon"
```

Delete all of the minikube clusters:

```shell
minikube delete --all
```


