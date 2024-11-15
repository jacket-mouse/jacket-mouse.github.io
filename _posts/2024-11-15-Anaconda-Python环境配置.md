---
layout: post
title: Anaconda-Python环境配置
---

## Anaconda 配置

---

Anaconda 是一个流行的 Python 和 R 数据科学平台，集成了包管理和环境管理功能。Anaconda 附带了 `conda` 包管理器，可以方便地管理 Python 环境和依赖。

### 安装 Anaconda

---

- 访问 [Anaconda 官网](https://www.anaconda.com/products/distribution)。

### 创建环境

---

```bash
conda create -n myenv python=3.8
```

此命令创建了一个名为 `myenv` 的虚拟环境，并指定 Python 版本为 3.8。

### 激活环境

---

```bash
conda activate myenv
```

在我的电脑上无法使用`conda activate`激活环境，于是使用下面的方案替代：

```bash
source ~/anaconda3/bin/activate ~/conda_envs/security_env
```

或者直接调用 `activate` 命令：

```bash
source activate ~/conda_envs/security_env
```

### 退出环境

```bash
conda deactivate
```

### 导出环境

```bash
conda env export > environment.yml
```

该命令会将当前环境的依赖写入 `environment.yml` 文件中，便于他人重现相同环境。

### 从文件导入环境

```bash
conda env create -f environment.yml
```

这会根据 `environment.yml` 文件安装相应的包和依赖，创建相同的环境。

### 列出所有环境

```bash
conda env list
```

### 常见 conda 命令总结

| 功能                  | 命令                                  |
| --------------------- | ------------------------------------- |
| 创建环境              | `conda create -n myenv python=3.8`    |
| 激活环境              | `conda activate myenv`                |
| 退出环境              | `conda deactivate`                    |
| 安装包                | `conda install numpy`                 |
| 更新包                | `conda update numpy`                  |
| 卸载包                | `conda remove numpy`                  |
| 导出环境              | `conda env export > environment.yml`  |
| 导入环境              | `conda env create -f environment.yml` |
| 列出所有环境          | `conda env list`                      |
| 删除环境              | `conda remove -n myenv --all`         |
| 启动 jupyter notebook | `jupyter notebook`                    |
