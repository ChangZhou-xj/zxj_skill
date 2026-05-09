# orange-aliyun 构建日志中 orange-patch 文件名提取

## 典型控制台输出（构建成功后）

### 1. orange-aliyun 成功日志（末尾片段）

```
----------> orange patch start
----------> 清理补丁包前端文件:orange-patch-20260507143455.zip
parseOrange -> ["parseOrange","orange-patch-20260507143455.zip","/home/jenkins/workspace/web/orange-aliyun/orange/dist/static/js/src/modules"]
处理前端其他文件
--------------------> 前端包临时文件 <--------------------

本次打包的前端版本文件:
vOrange_20260507143455

内网下载地址：http://192.168.4.22:8800/vOrange_20260507143455
外网下载地址：http://223.223.178.68:2004/orange-file/vOrange_20260507143455

生成的前端包:
orange_20260507143455.zip

内网下载地址：http://192.168.4.22:8800/orange_20260507143455.zip
外网下载地址：http://223.223.178.68:2004/orange-file/orange_20260507143455.zip

--------------------------------------------------------
<---------- orange patch end
Finished: SUCCESS
```

**关键提取规则**：
- `orange-patch-YYYYMMDDHHMMSS.zip` 出现在 `----------> 清理补丁包前端文件:` 之后
- 该文件名同时出现在 `parseOrange -> [...]` 数组的第二个元素中
- 用 `grep "orange-patch-" consoleText | tail -1` 可直接拿到

### 2. orange-patch 成功日志（末尾片段）

```
----------> orange_module   : pex
----------> orange_extra    : 
----------> orange build
----------> unzip orange zip
----------> orange module build orange_module_str: pex
----------> orange module build orange_extra_str: none
handleOrangeModule["handleOrangeModule","/home/jenkins/workspace/web/orange-patch/temp_2026-05-07-14-43-31/update_patch_pex_20260507144331/orange","pex","none"]
--------------------> 前端补丁包下载地址 <--------------------

文件名: update_patch_pex_20260507144331.zip

内网: http://192.168.4.22/jenkins-orange-patch/update_patch_pex_20260507144331.zip
外网: http://223.223.178.68:2004/jenkins-orange-patch/update_patch_pex_20260507144331.zip

vOrange

内网: http://192.168.4.22/jenkins-orange-patch/vOrange_pex_20260507144331
外网: http://223.223.178.68:2004/jenkins-orange-patch/vOrange_pex_20260507144331
-----------------------------------------------------------
----------> 清理旧前端包补丁:vOrange_orange_20260507130947
----------> 清理旧前端包补丁:update_patch_orange_20260507130947.zip
Finished: SUCCESS
```

**产物规律**：
- `update_patch_{orange_module}_{YYYYMMDDHHMMSS}.zip` — 补丁包
- `vOrange_{orange_module}_{YYYYMMDDHHMMSS}` — 完整模块包

## 从日志提取的 Python 代码

```python
import requests, re

JENKINS_URL = "http://223.223.178.68:2004/jenkins-122"
AUTH = ("develop", "dev1234")
BUILD_NUM = 30231  # orange-aliyun build number

session = requests.Session()
session.post(f"{JENKINS_URL}/j_spring_security_check",
    data={"j_username": "develop", "j_password": "dev1234", "from": "/"},
    auth=AUTH, timeout=30)

log = session.get(
    f"{JENKINS_URL}/job/web/job/orange-aliyun/{BUILD_NUM}/consoleText",
    auth=AUTH, timeout=30
).text

# 提取 orange-patch-YYYYMMDDHHMMSS.zip
m = re.search(r'orange-patch-(\d{14})\.zip', log)
if m:
    orange_patch_pkg = f"orange-patch-{m.group(1)}.zip"
    print(f"orange_package: {orange_patch_pkg}")
```

## 完整两阶段流程（已验证）

1. 触发 orange-aliyun → 从日志提取 `orange-patch-YYYYMMDDHHMMSS.zip`
2. 询问用户 `orange_module`（pex/home/orange）
3. 触发 orange-patch，参数：
   - `orange_package` = 上一步提取的文件名
   - `orange_module` = 用户指定

> **验证记录**：session #30231(orange-aliyun) → #7014(orange-patch pex) 全程 SUCCESS。
> **验证记录**：session #30278(orange-aliyun) → #7044(orange-patch pex) 全程 SUCCESS。

**orange-aliyun #30278** 产物（日志片段）：
```
vOrange_20260508173629
外网下载地址：http://223.223.178.68:2004/orange-file/vOrange_20260508173629
orange_20260508173629.zip
外网下载地址：http://223.223.178.68:2004/orange-file/orange_20260508173629.zip
parseOrange -> ["parseOrange","orange-patch-20260508173629.zip","..."]
```

**orange-patch #7044** 产物（日志片段）：
```
文件名: update_patch_pex_20260508174840.zip
外网: http://223.223.178.68:2004/jenkins-orange-patch/update_patch_pex_20260508174840.zip
vOrange_pex_20260508174840
外网: http://223.223.178.68:2004/jenkins-orange-patch/vOrange_pex_20260508174840
Finished: SUCCESS
```
