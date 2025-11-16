# 1. 使用支持 ARM64 的 Mamba 基础镜像 (Mamba 更快)
FROM condaforge/mambaforge:latest

# 2. 安装 wget，用于下载 Julia
# (mambaforge 镜像是 Debian 系，使用 apt)
USER root
RUN apt-get update && apt-get install -y wget && rm -rf /var/lib/apt/lists/*

# 3. 手动下载并安装 Julia for linux-aarch64 (ARM 64-bit)
# 我们把它安装到 /opt/ 目录下
RUN cd /opt && \
    wget https://julialang-s3.julialang.org/bin/linux/aarch64/1.10/julia-1.10.4-linux-aarch64.tar.gz && \
    tar -xzf julia-1.10.4-linux-aarch64.tar.gz && \
    rm julia-1.10.4-linux-aarch64.tar.gz

# 4. 将手动安装的 Julia 添加到系统 PATH 环境变量
# 这样 pyjulia (pysr的依赖) 才能找到它
ENV PATH="/opt/julia-1.10.4/bin:${PATH}"

# 5. 设置工作目录并复制所有文件
WORKDIR /app
COPY . .

# 6. 使用我们修改后的 environment.yml 文件创建环境
# 这会安装所有 Python 包，包括 pip 列表中的 pysr
RUN mamba env create -f docker/environment.yml

# 7. 激活环境，并预编译 pysr 的 Julia 依赖
# 这一步很重要，它会安装 SymbolicRegression.jl
SHELL ["conda", "run", "-n", "phyreg", "/bin/bash", "-c"]
RUN python -c "import pysr; pysr.install()"

# 8. 暴露端口并设置启动命令
EXPOSE 8888
CMD ["conda", "run", "-n", "phyreg", "jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root", "--LabApp.token=''"]