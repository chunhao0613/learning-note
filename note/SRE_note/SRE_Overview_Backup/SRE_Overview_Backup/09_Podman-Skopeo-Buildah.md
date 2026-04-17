# Podman-Skopeo-Buildah

[TOC]


---

## 認識與安裝 Skopeo


1. skopeo is a command line utility that performs various operations on container images and image repositories.
2. skopeo does not require the user to be running as root to do most of its operations.
3. skopeo does not require a daemon to be running to perform its operations.
4. skopeo can work with OCI images as well as the original Docker v2 images.


在 Alpine Linux 執行以下命令
```
$ sudo apk add skopeo

$ skopeo -v
skopeo version 1.8.0 commit: b7fc5b608b641cd9b9aec2647f9236e31f8f3b27
```


## Inspect an image in a remote registry


- skopeo 可以透過 `list-tags` / `inspect` 在還沒下載 image 下來之前，得知 image 的資訊

```bash=
$ sudo skopeo list-tags docker://docker.io/apache/ozone
{
    "Repository": "docker.io/apache/ozone",
    "Tags": [
        "0.3.0",
        "0.4.0",
        "0.4.1",
        "0.5.0",
        "1.0.0",
        "1.1.0",
        "1.2.0",
        "1.2.1",
        "latest"
    ]
}

$ sudo skopeo inspect docker://quay.io/cloudwalker/alpine
{
    "Name": "quay.io/cloudwalker/alpine",
    "Digest": "sha256:def822f9851ca422481ec6fee59a9966f12b351c62ccb9aca841526ffaa9f748",
    "RepoTags": [
        "latest"
    ],
    "Created": "2021-04-14T19:19:39.643236135Z",
    "DockerVersion": "19.03.12",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:540db60ca9383eac9e418f78490994d0af424aab7bf6d0e47ac8ed4e2e9bcbba"
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ]
}

```


---


## 下載並研究 Docker Image 內部結構


```bash=
$ sudo skopeo --insecure-policy copy docker://quay.io/cloudwalker/alpine  dir:/tmp/alpine
Getting image source signatures
Copying blob ca3cd42a7c95 done  
Copying config 49f356fa45 done  
Writing manifest to image destination
Storing signatures

$ tree /tmp/alpine
/tmp/alpine
├── 540db60ca9383eac9e418f78490994d0af424aab7bf6d0e47ac8ed4e2e9bcbba
├── 6dbb9cc54074106d46d4ccb330f2a40a682d49dda5f4844962b7dce9fe44aaec
├── manifest.json
└── version

* The Config and Image Layers are there, but remember we need to rely on a graph driver in a container engine to map them into a RootFS
```

檢視 Docker Image Manifest 資訊

```bash=
$ cat /tmp/alpine/manifest.json
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 1472,
      "digest": "sha256:6dbb9cc54074106d46d4ccb330f2a40a682d49dda5f4844962b7dce9fe44aaec"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 2811969,
         "digest": "sha256:540db60ca9383eac9e418f78490994d0af424aab7bf6d0e47ac8ed4e2e9bcbba"
      }
   ]
}
```

![](https://i.imgur.com/tHCcYXo.png)


檢視 Docker Image Layer 資訊

```bash=
$ cd /tmp/alpine; sudo mkdir img
 
$ sudo tar xvfz 59bf1c3509f33515622619af21ed55bbe26d24913cedbca106468a5fb37a50c3 -C img/

$ tree -L 1 img
img/
├── bin
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin
├── srv
├── sys
├── tmp
├── usr
└── var

17 directories, 0 files

$ cat 6dbb9cc54074106d46d4ccb330f2a40a682d49dda5f4844962b7dce9fe44aaec | jq
{
  "architecture": "amd64",
  "config": {
    "Hostname": "",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/bin/sh"
    ],
    "Image": "sha256:d3d4554f8b07cf59894bfb3551e10f89a559b24ee0992c4900c54175596b1389",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": null,
    "OnBuild": null,
    "Labels": null
  },
  "container": "60a3cdd128a8b373b313ed3e1083ff45e6badaad5dca5187282b005c38d04712",
  "container_config": {
    "Hostname": "60a3cdd128a8",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "Cmd": [
      "/bin/sh",
      "-c",
      "#(nop) ",
      "CMD [\"/bin/sh\"]"
    ],
    "Image": "sha256:d3d4554f8b07cf59894bfb3551e10f89a559b24ee0992c4900c54175596b1389",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": null,
    "OnBuild": null,
    "Labels": {}
  },
  "created": "2021-04-14T19:19:39.643236135Z",
  "docker_version": "19.03.12",
  "history": [
    {
      "created": "2021-04-14T19:19:39.267885491Z",
      "created_by": "/bin/sh -c #(nop) ADD file:8ec69d882e7f29f0652d537557160e638168550f738d0d49f90a7ef96bf31787 in / "
    },
    {
      "created": "2021-04-14T19:19:39.643236135Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"/bin/sh\"]",
      "empty_layer": true
    }
  ],
  "os": "linux",
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:b2d5eeeaba3a22b9b8aa97261957974a6bd65274ebd43e1d81d0a7b8b752b116"
    ]
  }
}
```


---


## 認識 OCI Image

This specification defines an OCI Image, consisting of a manifest, an image index (optional), a set of filesystem layers, and a configuration.

檢視 OCI Image 結構順序 
Image Index -> Image Manifest ->  Image Configuration + Filesystem Layer 


Runtime Filesystem bundle is what you get when you download the container image and unpack it.

OCI Image -> OCI Runtime Filesystem Bundle -> OCI Runtime (runc)



```bash=
$ sudo skopeo --insecure-policy copy docker://quay.io/cloudwalker/busybox:latest  oci:/home/bigred/busybox-oci:latest

$ cd; tree busybox-oci/
busybox_oci/
├── blobs
│   └── sha256
│       ├── 03781489f3738437ae98f13df5c28cc98bbc582254cfbf04cc7381f1c2ac1cc0
│       ├── 3cb635b06aa273034d7080e0242e4b6628c59347d6ddefff019bfd82f45aa7d5
│       └── bcd679c508ab0387148e6208479400b004f9bf9959124ce536b4be155f3251ca
├── index.json
└── oci-layout


* blob : Binary Large Object
```

- 透過 skopeo 看 image 可以得知 ， image 分為 docker image 以及 OCI image。


## 產生 Runtime Filesystem bundle


```
$ sudo apk add umoci

$ sudo umoci unpack --image busybox-oci:latest busybox-oci-bundle

$ sudo tree -L 2 busybox-oci-bundle/
busybox-oci-bundle/
├── config.json
├── rootfs
│   ├── bin
│   ├── dev
│   ├── etc
│   ├── home
│   ├── root
│   ├── tmp
│   ├── usr
│   └── var
├── sha256_67c32e0fe983b2f71469c2343e6747e3f664af16f1b414e14a70cbaabed53da6.mtree
└── umoci.json

9 directories, 3 files
```

- `umoci` ，可以幫我們把 rootfs 和 config.json 做整理，整理後就可以直接丟給 runc / crun / runsc 來使用



## Skopeo Syncing registries


```bash=
$ sudo skopeo login --tls-verify=false -u myquay -p myquay2022 $IP

將 Quay.io 中的 alpine image (包含所有 Tag) 同步到 自建的 Project Quay
$ sudo skopeo sync --tls-verify=false --src docker --dest docker quay.io/cloudwalker/alpine $IP/myquay

檢視同步到 自建 Project Quay 所有不同 Tag 的 alpine image  
$ sudo skopeo list-tags --tls-verify=false docker://$IP/myquay/alpine
{
    "Repository": "192.168.61.149/myquay/alpine",
    "Tags": [
        "latest"
    ]
}

```


## YAML file content (sync.yml)


```bash=
$ nano sync.yaml
quay.io:
    images:
        cloudwalker/busybox: []
        cloudwalker/alpine.sshd:
            - latest
        cloudwalker/alpine.derby:
            - latest

$ sudo skopeo sync --tls-verify=false --src yaml --dest docker sync.yaml $IP/myquay/

$ sudo skopeo list-tags --tls-verify=false docker://$IP/myquay/busybox
{
    "Repository": "192.168.61.149/myquay/busybox",
    "Tags": [
        "1.35.0",
        "latest"
    ]
}

```


---


# 探索 Buildah


```bash=
$ mkdir ~/alpine.sshd; cd ~/alpine.sshd  
$ nano Dockerfile
FROM 192.168.61.149/myquay/alpine
RUN \
  apk update && \
  apk add --no-cache nano sudo bash wget curl git tree grep \
    elinks shadow procps util-linux coreutils binutils \
    findutils openssh-server tzdata && \
  # 設定時區
  cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime && \
  ssh-keygen -t rsa -P "" -f /etc/ssh/ssh_host_rsa_key && \
  echo -e 'Welcome to ALP sshd 6000\n' > /etc/motd && \
  # 建立管理者帳號 bigred
  adduser -s /bin/bash -h /home/bigred -G wheel -D bigred && \
  echo "%wheel   ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
  echo -e 'bigred\nbigred\n' | passwd bigred &>/dev/null && \
  rm /sbin/reboot && rm /usr/bin/killall
ENTRYPOINT ["/usr/sbin/sshd"]
CMD ["-D"]
```

```bash=
$ buildah bud --tls-verify=false --format=docker --squash -t alp.sshd .
```

* `--squash` 這參數在 image 中只產生一個 資料目錄 (layer), 但會保留 image 製作過程, 也就是 Dockerfile 內容，如果沒有加這個參數，Dockerfile 裡面有幾個參數，就會有幾個壓縮檔


```
$ buildah images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
localhost/alp.sshd  latest  f662096fabbe   6 minutes ago   62.2 MB

$ podman run --name s1 -d -p 22100:22 localhost/alp.sshd
00125e52bcb7f56ef74d63a8937a67c41bd00f16c0f21eaf37092b8ee0598505

$ ssh bigred@localhost -p 22100
bigred@localhost's password: bigred
Welcome to ALP sshd 6000

00125e52bcb7:~$ exit
```


## Building a Image


```bash=
$ container=$(buildah from alpine)
$ echo $container
alpine-working-container

$ buildah run $container -- apk add bash
$ buildah run $container -- apk add --update nodejs nodejs-npm

# 在外面設定 Dockerfile 的命令
$ buildah config --workingdir /usr/src/app/ $container
$ buildah run $container -- npm init -y
Wrote to /usr/src/app/package.json:
{
  "name": "app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```


## Building a Image


```bash=
$ nano HelloWorld.js
const express = require('express')
const app = express()
const port = 3000
app.get('/', (req, res) => res.send('Hello World!'))
app.listen(port, () => console.log(`Example app listening on port ${port}!`))

$ buildah copy $container HelloWorld.js

$ buildah config --entrypoint "node HelloWorld.js" $container
WARN[0000] cmd "/bin/sh" exists but will be ignored because of entrypoint settings

# 將 $container 做成 image 
$ buildah commit $container buildah-hello-world

$ buildah images
REPOSITORY                      TAG                IMAGE ID       CREATED          SIZE
localhost/buildah-hello-world   latest             b685e41a1079   6 seconds ago    69.6 MB
```



###### tags: `系統工程`
























