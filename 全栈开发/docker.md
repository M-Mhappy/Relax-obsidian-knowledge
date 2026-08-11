### 马上能动手的最小实践：把你之前写的 `medapi.py` 容器化

**目标**：把最基础的 `FastAPI` 应用打包成一个 Docker 镜像，并跑起来。这是地基，我们分四步走。

**准备工作**：  
你需要安装 Docker Desktop。去 [docker.com](https://www.docker.com/products/docker-desktop/) 下载并安装。安装好后，打开终端运行 `docker --version` 确认一下。

---

**第一步：准备你的应用代码**

创建一个新目录 `ai-docker-demo`，然后把我们最早写的 `medapi.py` 复制进去。

python
from fastapi import FastAPI
app = FastAPI()
@app.get("/health")
async def health_check():
    return {"status": "healthy", "message": "Dockerized!"}

同时，在这个目录下创建一个 `requirements.txt`：
fastapi==0.115.6
uvicorn==0.34.0

（版本号可以不加，但加上是生产环境的好习惯）

---

**第二步：编写“集装箱图纸”——Dockerfile**

在 `ai-docker-demo` 目录下，新建一个文件，名字就叫 `Dockerfile`（没有后缀名）。内容如下：

dockerfile
#1. 基础镜像：我们用一个已经装好 Python 3.11 的轻量集装箱做地基
FROM python:3.11-slim
#2. 设置工作目录：相当于集装箱里的“cd /app”，所有后续操作都在这个目录下
WORKDIR /app
#3. 复制依赖文件并安装：先只复制 requirements.txt，这样代码没变时，Docker能用缓存加速构建
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
#4. 复制应用代码：把当前目录下的所有文件，复制到集装箱的 /app 目录里
COPY . .
#5. 告诉 Docker，这个集装箱会通过 8000 端口提供服务（文档性质，不做实际映射）
EXPOSE 8000
 #6. 容器启动时，立刻执行这个命令来跑我们的服务
CMD ["uvicorn", "medapi:app", "--host", "0.0.0.0", "--port", "8000"]

**注意**：`--host 0.0.0.0` 是必须的，它让容器内的服务可以接受来自宿主机和其他容器的连接。

---
**第三步：建造集装箱——构建镜像**

在 `ai-docker-demo` 目录下，打开终端，运行：

bash
docker build -t my-medapi:v1 .
- `-t my-medapi:v1`：给镜像打标签（名字 + 版本）。

- `.`：告诉 Docker，Dockerfile 在当前目录。

你会看到 Docker 一步步执行 `Dockerfile` 里的指令。第一次会慢一点。

---

**第四步：启动集装箱——运行容器**

bash

docker run -d -p 8000:8000 --name my-medapi-container my-medapi:v1

- `-d`：后台运行。
    
- `-p 8000:8000`：**端口映射**。把宿主机的 8000 端口，和容器的 8000 端口打通。
    
- `--name`：给容器起个名字。

现在，打开浏览器访问 `http://localhost:8000/health`，看到熟悉的 JSON 返回。你的应用现在已经跑在 Docker 集装箱里了。你可以用 `docker ps` 看到正在运行的容器，用 `docker stop my-medapi-container` 停掉它。

docker-compose.yml配置中启动 Volume时，相关容器存在Volume，不随容器删除而丢失，切换容器版本一般也不影响；未启动时，数据绑定容器层，随容器删除会丢失。
当容器发生大版本更新迭代时，前后版本可能存在数据格式不兼容，必须先备份数据再升级容器，然后删除旧版本，拉起新版本，导入旧数据。
"旧数据不兼容"指的是二进制数据文件不兼容，但我们可以通过 `pg_dump` 把数据转换成通用的 SQL 文本，再导入到新版本中。这不是"恢复旧数据"，而是"用新版本重建数据"。