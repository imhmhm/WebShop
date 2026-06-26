# WebShop -- WebStudio 环境搭建指南

既然您都看到这了，说明您已经知道 WebStudio 是什么，并且需要在 WebStudio 环境搭建 WebShop

> ⚠️ 前提：
> 1. 您使用了`ailab-910b-pt_2.9.0-sgl_0.5.9-vllm_0.18.0-cann_9.0.0-py_3.11:25.6.1.401`镜像启动 WebStudio
> 2. 脚本中多处标注 `[webstudio]` 的步骤，是因为 WebStudio 环境无法访问 Google Drive、无法在线安装 spaCy 语言包等，可以从华山平台复制离线数据/包，具体地址私聊获取
> 3. 虽然镜像内默认conda环境名是`PyTorch-2.7.1`（基础镜像自带），但实际已经升级`torch==2.9.0`，（历史问题没有修改，此处不赘述原因）
---

## 1. 修改初始化脚本 `init_mtp.sh`

备份原文件并删除其中自动激活 conda 环境的行（避免环境冲突）。

```bash
sed -i.bak '/conda activate \$ENV_NAME/d' /home/ma-user/init_mtp.sh
```

## 2. 创建 conda 环境

基于已有的 `PyTorch-2.7.1` 环境 clone 名为 `webshop` 的新环境。

```bash
conda create --name webshop --clone PyTorch-2.7.1
conda activate webshop
```

## 3. 安装依赖

### 3.1 Python 依赖

```bash
git clone -b webstudio https://github.com/imhmhm/WebShop.git
cd WebShop
pip install -r requirements_webstudio.txt
```

### 3.2 OpenJDK 21

通过 conda-forge 安装（pyserini 依赖 Java）。

```bash
conda install -c conda-forge openjdk=21
```

## 4. 修复 pyserini 1.4.0 的 bug

> `pyserini==1.4.0` 是最后一个支持 `numpy<2` 的版本，但存在一个 bug，按如下方式删除 `_openai.py` 中第 27–30 行即可修复。

```bash
sed -i.bak '27,30d' /home/ma-user/anaconda3/envs/webshop/lib/python3.11/site-packages/pyserini/encode/_openai.py
```

## 5. 准备数据集（从华山平台复制）

> `[webstudio]` Google Drive 无法连接，可以从华山平台复制相关数据到本地 `data/` 目录。

```bash
mkdir -p data
cd data
# 需要从华山平台复制以下文件到当前目录：
#   - items_shuffle_1000
#   - items_ins_v2_1000
#   - items_shuffle
#   - items_ins_v2
#   - items_human_ins
cd ..
```

## 6. 安装 spaCy 语言包（离线）

> `[webstudio]` spaCy 无法在线安装语言包 `en_core_web_lg` 和/或 `en_core_web_sm`，可以从华山平台复制对应 whl 包后离线安装。

```bash
# 从华山平台复制以下 whl 文件：
#   - en_core_web_lg-*.whl
#   - en_core_web_sm-*.whl
# 然后离线安装：
pip install en_core_web_lg-*.whl
pip install en_core_web_sm-*.whl
```

## 7. 构建搜索引擎索引

```bash
cd search_engine
mkdir -p resources resources_100 resources_1k resources_100k
python convert_product_file_format.py   # 将 items.json 转换为所需的文档格式
mkdir -p indexes
./run_indexing.sh
cd ..
```

## 8. 准备用户会话日志（从华山平台复制）

> `[webstudio]` Google Drive 无法连接，可以从华山平台复制 `all_trajs.zip` 到本地。

```bash
mkdir -p user_session_logs/
cd user_session_logs/
# 从华山平台复制 all_trajs.zip 到当前目录
# unzip all_trajs.zip
# rm all_trajs.zip
cd ..
```
