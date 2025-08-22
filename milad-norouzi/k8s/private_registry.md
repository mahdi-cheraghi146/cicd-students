# Configuring containerd to use a private registry

With containerd, you do not use docker login.
Instead, you create or edit an auth config file (usually at /etc/containerd/certs.d/<registry>/hosts.toml) to store your credentials.

Example: Login to image registry with containerd
Create the directory for your registry:

Edit the containerd configuration in /etc/containerd/config.toml (i.e., modify the line marked adding the path indicated):

```bash
sudo vim /etc/containerd/config.toml
```

Add the following lines to the file:

```toml
[plugins."io.containerd.grpc.v1.cri".registry]

      config_path = "/etc/containerd/certs.d"
```


Then create the directory for your registry:

```bash
sudo mkdir -p /etc/containerd/certs.d/docker.anisa.local

sudo vim /etc/containerd/certs.d/docker.anisa.local/hosts.toml
```

Generate a base64-encoded username:password string for authentication. You can use the following command:

```bash
echo -n 'username:password' | base64

echo -n "admin:Aa123456" | base64
YWRtaW46QWExMjM0NTY=
```

### Add the following content to the hosts.toml file

```toml
server = "http://docker.anisa.local"

[host."http://docker.anisa.local"]
  capabilities = ["pull", "resolve", "push"]
  skip_verify = true

  [host."http://docker.anisa.local".auth]
    authentication = "Basic YWRtaW46QWExMjM0NTY=" # <base64-encoded-username:password> 
```


### Restart containerd to apply the changes

```bash
sudo systemctl restart containerd
```

### Verify the configuration

```bash
ctr image pull docker.anisa.local/nginx:1.0.0
ctr image push docker.anisa.local/nginx:1.0.0
```

Refrences:
https://github.com/containerd/containerd/discussions/6468

https://gardener.cloud/docs/gardener/advanced/containerd-registry-configuration/

https://aams-eam.pages.dev/posts/deploying-a-kubernetes-cluster-with-containerd-and-an-insecure-private-docker-registry/
