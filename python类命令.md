# conda



## 设置镜像

### conda镜像设置

**查看conda环境配置**：`conda config --show`

**清华源**

```bash
# 删除配置的镜像
conda config --remove-key channels

# 查看当前配置
conda config --show

# 添加主仓库
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
# 添加免费仓库
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
# 添加社区仓库 (替代旧的 cloud/conda-forge)
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
# (可选) 如果你需要 R 语言包
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r/

# 设置搜索时显示通道地址
conda config --set show_channel_urls yes

# 删除默认源（可选）
conda config --remove channels defaults
```

**中科大镜像**

```bash
# 中科大镜像源（备用）
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
```

**临时使用**

```bash
conda install package_name -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
```

**恢复默认源**

```bash
conda config --remove-key channels
```

**配置文件位置**

- Linux/Mac: `~/.condarc`
- Windows: `C:\Users\<用户名>\.condarc`



### pip镜像配置

**使用临时镜像**

```bash
pip install package_name -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**永久配置**:

**方法一：配置文件（推荐）**

bash

```bash
# Linux/Mac
mkdir -p ~/.pip
vim ~/.pip/pip.conf

# Windows
# 在用户目录（如：C:\Users\用户名）创建 pip 文件夹
# 在 pip 文件夹中创建 pip.ini 文件
```

**配置文件内容：**

```bash
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn

# 或者使用阿里云镜像
# index-url = https://mirrors.aliyun.com/pypi/simple/
# trusted-host = mirrors.aliyun.com
```



**方法二：命令行设置**

```bash
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip config set install.trusted-host pypi.tuna.tsinghua.edu.cn
```

**常用国内镜像**

```bash
# 清华
https://pypi.tuna.tsinghua.edu.cn/simple

# 阿里云
https://mirrors.aliyun.com/pypi/simple/

# 腾讯云
https://mirrors.cloud.tencent.com/pypi/simple

# 华为云
https://mirrors.huaweicloud.com/repository/pypi/simple

# 豆瓣
https://pypi.douban.com/simple/

# 中科大
https://pypi.mirrors.ustc.edu.cn/simple/
```

**查看配置**

```bash
# Conda
conda config --show

# Pip
pip config list
```





## 常用命令

- 查看conda版本：`conda --version`
- 更新conda：`conda update conda`
- 更新Anaconda整体：`conda update Anaconda`
- 查看某个命令的帮助：`conda create --help`

### 1 管理环境

#### **创建虚拟环境：**

```bash
conda create -n env_name python=3.8
```

这表示创建python版本为3.8、名字为env_name的虚拟环境。

创建后，env_name文件可以在Anaconda安装目录envs文件下找到。在不指定python版本时，自动创建基于最新python版本的虚拟环境.   

#### 查看有哪些虚拟环境

以下三条命令都可以。注意最后一个是”--”，而不是“-”.

```bash
conda env list
conda info -e
conda info --envs
```

#### 激活虚拟环境

```bash
conda activate env_name
```

此时使用`python --version`可以检查当前python版本是否为所想要的（即虚拟环境的python版本）。

#### 退出虚拟环境

```bash
conda activate
conda deactivate
```

有意思的是，以上两条命令只中任一条都会让你回到base environment，它们从不同的角度出发到达了同一个目的地。可以这样理解，activate的缺省值是base，deactivate的缺省值是当前环境，因此它们最终的结果都是回到base

#### 删除虚拟环境

执行以下命令可以将该指定虚拟环境及其中所安装的包都删除。

```bash
conda remove --name env_name --all
```

如果只删除虚拟环境中的某个或者某些包则是：

```bash
conda remove --name env_name  package_name
```

#### 导出环境

很多的软件依赖特定的环境，我们可以导出环境，这样方便自己在需要时恢复环境，也可以提供给别人用于创建完全相同的环境。

```bash
#获得环境中的所有配置
conda env export --name myenv > myenv.yml
#重新还原环境
conda env create -f  myenv.yml
```

### 2 包管理

#### 查询包的安装情况

查询看当前环境中安装了哪些包：`conda list`

查询当前Anaconda repository中是否有你想要安装的包：`conda search package_name`

**查询是否有安装某个包**:
用conda list后跟package名来查找某个指定的包是否已安装，而且支持*通配模糊查找。

```bash
conda list pkgname        
conda list pkgname*       
```

#### 包的安装和更新

在当前（虚拟）环境中安装一个包：`conda install package_name`

安装某个特定版本的包：`conda install numpy=0.20.3`

将某个包更新到它的最新版本 ：`conda update numpy`

安装包的时候可以指定从哪个channel进行安装，比如说，以下命令表示不是从缺省通道，而是从conda_forge安装某个包:
```bash
conda install pkg_name -c conda_forge
```

**从requirements.txt安装**

```bash
pip install -r requirements.txt
```

#### conda卸载包

```bash
conda uninstall package_name
```

这样会将依赖于这个包的所有其它包也同时卸载。

如果不想删除依赖其当前要删除的包的其他包：

```bash
conda uninstall package_name --force
```

 但是并不建议用这种方式卸载，因为这样会使得你的环境支离破碎，

####  清理anaconda缓存

```bash
conda clean -p      # 删除没有用的包 --packages
conda clean -t      # 删除tar打包 --tarballs
conda clean -y -all # 删除所有的安装包及cache(索引缓存、锁定文件、未使用过的包和tar包)
```

conda就像个守财奴一样，把每个历史安装包都会好好保存。。。好处是可以很方便地恢复到旧的历史版本，坏处是占内存空间。。。

### 3 Python版本的管理

#### 将版本变更到指定版本

```bash
conda install python=3.5
python --version
```

#### 将python版本更新到最新版本

```bash
conda update python
```

### conda install vs pip install

#### 有什么区别？

1. conda可以管理非python包，pip只能管理python包。
2. conda自己可以用来创建环境，pip不能，需要依赖virtualenv之类的。
3. conda安装的包是编译好的二进制文件，安装包文件过程中会自动安装依赖包；pip安装的包是wheel或源码，装过程中不会去支持python语言之外的依赖项。
4. conda安装的包会统一下载到一个目录文件中，当环境B需要下载的包，之前其他环境安装过，就只需要把之间下载的文件复制到环境B中，下载一次多次安装。pip是直接下载到对应环境中。
5. conda只能在conda管理的环境中使用，例如比如conda所创建的虚环境中使用。pip可以在任何环境中使用，在conda创建的环境 中使用pip命令，需要先安装pip（conda install pip ），然后可以 环境A 中使用pip 。conda 安装的包，pip可以卸载，但不能卸载依赖包，pip安装的包，只能用pip卸载。

#### 安装在哪里？

- conda install xxx：这种方式安装的库都会放在anaconda3/pkgs目录下，这样的好处就是，当在某个环境下已经下载好了某个库，再在另一个环境中还需要这个库时，就可以直接从pkgs目录下将该库复制至新环境而不用重复下载。
  ```bash
  # 查看conda环境路径
  conda info
  
  # 默认位置：
  # Linux/Mac: ~/anaconda3/envs/<环境名>/lib/python3.x/site-packages/
  # Windows: C:\Users\<用户名>\anaconda3\envs\<环境名>\Lib\site-packages\
  
  # 基础环境位置：
  # ~/anaconda3/lib/python3.x/site-packages/  (base环境)
  ```
  
  ```bash
  # 查看pip安装位置
  pip show <包名> | grep Location
  # 或
  python -m site
  
  # 系统Python位置：
  # Linux/Mac: /usr/local/lib/python3.x/site-packages/
  # Windows: C:\Python3x\Lib\site-packages\
  
  # 用户级别位置：
  # Linux/Mac: ~/.local/lib/python3.x/site-packages/
  # Windows: C:\Users\<用户名>\AppData\Roaming\Python\Python3x\site-packages\
  ```
  
- pip install xxx：分两种情况，一种情况就是当前conda环境的python是conda安装的，和系统的不一样，那么xxx会被安装到anaconda3/envs/current_env/lib/python3.x/site-packages文件夹中，如果当前conda环境用的是系统的python，那么xxx会通常会被安装到~/.local/lib/python3.x/site-packages文件夹中 

#### 如何判断conda中某个包是通过conda还是pip安装的？

行 `conda list` ，用pip安装的包显示的build项目为pypi





---



# uv

## 从空文件开始

从空文件夹开始，使用 `uv` 管理 Python 项目，可以遵循一个清晰、标准化的流程。

`uv` 的强大之处在于，它能将**项目初始化、依赖管理、虚拟环境、运行脚本**等环节全部串联起来，让你只需几个命令就能完成从零到一的搭建。

下面是一个标准的、从空文件夹开始的完整工作流：

### 📂 第一步：安装与初始化项目

首先，确保你已经安装了 `uv`。如果还没有，可以通过 `pip` 或 `pipx` 安装。

```bash
# 通过 pip 安装
pip install uv
```

接下来，我们开始创建一个新项目：

1.  **创建项目目录**：首先，创建一个新的空文件夹并进入。
    ```bash
    mkdir my-awesome-project
    cd my-awesome-project
    ```

2.  **初始化项目**：在空文件夹中运行 `uv init` 命令。这会生成一个 Python 项目所需的基础文件。
    ```bash
    uv init
    ```
    执行后，`uv` 会创建以下几个核心文件:
    *   `pyproject.toml`：项目的核心配置文件，所有依赖和元数据都在这里定义。
    *   `main.py`：一个包含 "Hello World" 的示例 Python 文件。
    *   `README.md`：项目的说明文档。
    *   `.python-version`：指定项目使用的 Python 版本。

    如果你需要创建一个更规范的、可发布的包（例如将代码放在 `src/` 目录下），可以使用 `--package` 参数：
    ```bash
    uv init --package
    ```
    
    如果你需要创建一个更简单干净的环境，可以使用 `--no--package` 参数：
    
    ```bash
    uv init --no--package
    ```
    
    

### ⚙️ 第二步：管理依赖与环境

这是 `uv` 最核心的环节。它极大地简化了依赖和虚拟环境的管理。

1.  **添加依赖**：使用 `uv add` 命令来安装项目依赖。`uv` 会自动将依赖添加到 `pyproject.toml` 文件中，并更新锁文件。
    ```bash
    # 添加 requests 库作为项目依赖
    uv add requests
    ```
    你也可以添加仅在开发时需要的依赖（如 `pytest`, `ruff`, `mypy`）：
    ```bash
    # 添加 ruff 作为开发依赖
    uv add --dev ruff
    ```
    `--dev` 参数会将依赖添加到 `[tool.uv.dev-dependencies]` 部分。

2.  **同步环境**：`uv sync` 命令会根据 `pyproject.toml` 和 `uv.lock` 文件，自动创建一个 `.venv` 虚拟环境，并安装所有依赖。这是确保环境与项目配置一致的最佳实践。
    ```bash
    uv sync
    ```

### 🚀 第三步：运行项目与代码

现在，你的项目环境已经就绪，可以开始编码和运行了。

1.  **运行脚本**：使用 `uv run` 来执行 Python 文件。它会自动在项目的虚拟环境中运行你的代码，无需手动激活环境。
    ```bash
    # 运行默认生成的 main.py
    uv run main.py
    ```

2.  **运行其他命令**：`uv run` 也可以用来执行环境中的其他工具，比如运行测试。
    ```bash
    # 假设你已经通过 uv add --dev pytest 安装了 pytest
    uv run pytest
    ```

3.  **进入交互式环境**：你也可以用 `uv run` 启动一个 Python 交互式 shell，方便进行调试。
    ```bash
    uv run python
    ```

### 💎 总结

总结一下，从空文件夹开始使用 `uv` 的完整命令序列就是：
```bash
# 1. 创建并进入项目目录
mkdir my-project && cd my-project

# 2. 初始化项目
uv init

# 3. 添加依赖 (例如 requests)
uv add requests

# 4. （可选）添加开发依赖
uv add --dev pytest

# 5. 同步并创建虚拟环境，安装所有依赖
uv sync

# 6. 运行你的代码
uv run main.py
```

通过这几步，你就拥有了一个结构清晰、依赖明确、环境隔离的现代化 Python 项目。`uv` 将原本繁琐的环境配置工作变得异常简单高效。







## 常用命令



**创建虚拟环境**

```bash
uv venv
```

**安装依赖**

```bash 
uv add package # 若该包依赖其他包会一同安装
```

安装会自动修改`pyproject.py`和`uv.lock`

> pyproject.py :
>
> 自动添加dependencies

> uv.lock 自动写入 无需手动修改



**查看当前环境依赖**

```bash
uv tree
```



**运行文件**

```bash
uv run main.py
```

uv会使用当前项目的虚拟环境运行py文件



**项目分发**

```bash
uv sync
```

clone项目拿到手后，在项目目录运行该命令就会根据`pyproject.py`和`uv.lock`创建虚拟环境



**删除依赖**

```bash
uv remove 包名
```



**安装指定python**

```bash
uv python install cpython3.12
```

**指定项目python版本号**

```python
uv python pin 3.12
```

**查看已安装和未安装的python版本**

```python
uv python list
```

**更新自身**

```bash
uv self update
```

**卸载**

```bash
uv uv self uvinstall
```

**安装uv配套的工具**

```python
uv tool install ruff
```

**用工具检查语法格式以及代码问题**

```bash
ruff check main.py
```











