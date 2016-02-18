---
layout: post
title: "Docker Log"
---

```bash
docker run -it centos bash
```

Chạy test docker, nếu không tìm thấy local image sẽ download trên kho. Sau khi chạy sẽ là shell của centos.
Sau khi custom image, commit để lưu thay đổi.

```bash
docker commit -m "commit message" -a "author" commit_id image_name:tag
```

## Công nghệ bên dưới docker

### Namespace

### Control groups (cgroups)

### Union file systems

### Container format

## Facts

Docker được viết bằng Go

## Tham khảo

1. <https://docs.docker.com/engine/userguide/containers/dockerimages/>
2. <https://stackoverflow.com/questions/22907231/copying-files-from-host-to-docker-container>
