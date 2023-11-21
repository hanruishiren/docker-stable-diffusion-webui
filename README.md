# Harry的提示：
要生成一个Docker镜像，你需要使用`docker build`命令，该命令的基本语法如下：

```bash
docker build -t <镜像名称>:<标签> <路径>
```

其中：
- `<镜像名称>` 是你想要给镜像起的名称。
- `<标签>` 是你想要给镜像打的标签，通常是版本号或者其他标识。
- `<路径>` 是包含 Dockerfile 的目录路径。

例如，如果你的Dockerfile在当前目录下，你可以运行如下命令：

```bash
docker build -t myimage:1.0 .
```

这个命令会在当前目录下查找Dockerfile，并使用它构建一个名为`myimage`，标签为`1.0`的Docker镜像。

生成的镜像默认是存储在本地的 Docker 主机上。如果你想要将镜像推送到 Docker Hub 或其他容器注册表，你需要使用 `docker push` 命令。首先，你需要登录到 Docker Hub，然后运行以下命令：

```bash
docker login
```

然后，使用以下命令将镜像推送到 Docker Hub：

```bash
docker push <镜像名称>:<标签>
```

例如：

```bash
docker push myimage:1.0
```

这样就会将 `myimage` 镜像的 `1.0` 版本推送到 Docker Hub。确保你有相应的权限来推送到指定的容器注册表。

---
# 原文readme如下：


# Stable Diffusion web UI
A browser interface based on Gradio library for Stable Diffusion.

## Usage

Basic example:
```
docker run --gpus all --restart unless-stopped -p 8080:8080 \
  --name stable-diffusion-webui -d universonic/stable-diffusion-webui
```

### Where to Store Data

It is recommended to create a data directory on the host system (outside the container) and mount this to a directory visible from inside the container. This places the data files (including: extensions, models, outputs, etc.) in a known location on the host system, and makes it easy for tools and applications on the host system to access the files. The downside is that the user needs to make sure that the directory exists, and that e.g. directory permissions and other security mechanisms on the host system are set up correctly. 

We will simply show the basic procedure here:
1. Create a data directory on a suitable volume on your host system, e.g. `/my/own/datadir`.
2. Start your `stable-diffusion-webui` container like this:
```
docker run --gpus all --restart unless-stopped -p 8080:8080 \
  -v /my/own/datadir/extensions:/app/stable-diffusion-webui/extensions \
  -v /my/own/datadir/models:/app/stable-diffusion-webui/models \
  -v /my/own/datadir/outputs:/app/stable-diffusion-webui/outputs \
  -v /my/own/datadir/localizations:/app/stable-diffusion-webui/localizations \
  --name stable-diffusion-webui -d universonic/stable-diffusion-webui
```
3. Troubleshooting with the following command if you encountered problems:
```
docker logs -f stable-diffusion-webui
```

**Using docker-compose**:
```
version: "3.2"

services:
  stable-diffusion-webui:
    image: universonic/stable-diffusion-webui:minimal
    command: --no-half --no-half-vae --precision full
    runtime: nvidia
    restart: unless-stopped
    ports:
      - "8080:8080/tcp"
    volumes:
      - /my/own/datadir/inputs:/app/stable-diffusion-webui/inputs
      - /my/own/datadir/textual_inversion_templates:/app/stable-diffusion-webui/textual_inversion_templates
      - /my/own/datadir/embeddings:/app/stable-diffusion-webui/embeddings
      - /my/own/datadir/extensions:/app/stable-diffusion-webui/extensions
      - /my/own/datadir/models:/app/stable-diffusion-webui/models
      - /my/own/datadir/localizations:/app/stable-diffusion-webui/localizations
      - /my/own/datadir/outputs:/app/stable-diffusion-webui/outputs
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    deploy:
      mode: global
      placement:
        constraints:
          - "node.labels.iface != extern"
      restart_policy:
        condition: unless-stopped
      resources:
        reservations:
          devices:
            - driver: nvidia
              capabilities: [gpu]
```

**Important note**: `stable-diffusion-webui` will not successfully start and keep restarting if you have no Stable Diffusion model files available in the model directory. e.g. `/my/own/datadir/models/Stable-diffusion`. Put checkpoint and vae file in that directory, wait the server to start and enjoy yourself now.

Follow the [official instructions](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki) if you have further questions.
