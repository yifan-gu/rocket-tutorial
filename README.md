# Rocket Tutorial

```
package main

import (
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        log.Printf("request from %v\n", r.RemoteAddr)
        w.Write([]byte("hello\n"))
    })
    log.Fatal(http.ListenAndServe(":5000", nil))
}
```

```
CGO_ENABLED=0 GOOS=linux go build -a -tags netgo -ldflags '-w' .
```

```
{
    "acVersion": "0.1.1",
    "acKind": "ImageManifest",
    "name": "coreos.com/hello-1.0.0",
    "app": {
        "exec": [
            "/bin/hello"
        ]
    },
    "annotations": {
        "authors": "Kelsey Hightower <kelsey.hightower@gmail.com>"
    }
}
```

```
actool validate hello-manifest
```

```
mkdir -p hello-app/rootfs/bin
```

```
cp hello-manifest hello-app/manifest
cp hello hello-app/rootfs/bin/
```

```
actool build hello-app hello.aci
```

```
actool validate hello.aci
```

### Launch a local application image

```
sudo rkt run hello.aci
```

### Testing with curl

```
curl 127.0.0.1:5000
```

## Hosting App Container Images

### Install a webserver

```
brew install nginx
nginx
```

### Copy the binary

```
scp core@192.168.12.138:~/hello.aci .
mv hello.aci /usr/local/var/www/
```

### Run it.

```
sudo rkt run http://192.168.12.1:8080/hello.aci
```

## Launch a Docker Container with Rocket

### Download and export a Docker Container

```
docker pull coreos/etcd
```

```
docker run --name=etcd coreos/etcd
```

### Export the Docker Container

```
mkdir -p etcd-app/rootfs
```

```
docker export etcd | sudo tar -x -C etcd-app/rootfs -f -
```

### Create the app manifest

```
{
    "acVersion": "0.1.1",
    "acKind": "ImageManifest",
    "name": "coreos.com/etcd",
    "app": {
        "exec": [
            "/etcd -name node0"
        ]
    },
    "annotations": {
        "authors": "Kelsey Hightower <kelsey.hightower@gmail.com>"
    }
}
```

```
cp etcd-manifest etcd-app/manifest
```

### Build the container

```
actool build etcd-app etcd.aci
```

### Launch the container with Rocket

```
sudo rkt run etcd.aci
```
