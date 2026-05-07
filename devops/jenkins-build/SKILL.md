---
name: jenkins-build
description: 使用 Jenkins 进行构建、打包、触发流水线任务。触发流水线、下载制品、解析构建日志。
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [jenkins, ci-cd, build, 打包, devops]
    related_skills: [github-pr-workflow]
---

# Jenkins 构建与打包

## 概述

通过 Jenkins REST API 或 CLI 触发构建任务、下载制品（JAR/WAR/ZIP/二进制文件）、解析控制台输出、管理流水线执行。适用于标准 Jenkins（Jenkinsfile 流水线）和 Blue Ocean UI。

## 何时使用

- 通过 API 触发 Jenkins 构建/任务
- 下载构建制品（JAR、WAR、ZIP、二进制文件等）
- 解析构建控制台输出，判断成功/失败/状态
- 轮询构建状态直到完成
- 使用 Jenkins CLI（jenkins-cli.jar）进行高级操作
- 通过 Jenkins 触发 Maven/Gradle 打包
- 获取构建参数或上次失败原因

## 前置条件

- Jenkins 地址（如 `http://223.223.178.68:2004/jenkins-122/`）
- 有 API 权限的 Jenkins 用户及 API Token
- 任务名称及（可选）构建参数

### 默认配置（当前环境）

```bash
export JENKINS_URL="http://223.223.178.68:2004/jenkins-122"
export JENKINS_USER="develop"
export JENKINS_TOKEN="dev1234"
# 前端打包任务路径：job/web/job/orange-aliyun/build
```

### 获取 Jenkins API Token

1. 登录 Jenkins Web UI
2. 点击用户名 → **Configure** → **API Token** → **Add new Token**
3. 复制 Token（当作密码处理）

## 常用操作

### 1. 触发构建（REST API）

```bash
# 简单构建（无参数）
curl -s -X POST \
  "$JENKINS_URL/job/JOB_NAME/build" \
  -u "$JENKINS_USER:$JENKINS_TOKEN"

# 带参数构建（POST 表单）
curl -s -X POST \
  "$JENKINS_URL/job/JOB_NAME/buildWithParameters" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "PARAM1=value1&PARAM2=value2"

# 立即加入队列（不等待）
curl -s -X POST \
  "$JENKINS_URL/job/JOB_NAME/build" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" \
  --data-urlencode ""
```

### 2. 获取构建状态

```bash
# 最新构建号 + 结果
curl -s "$JENKINS_URL/job/$JOB_NAME/lastBuild/api/json" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | jq '{number, result, duration}'

# 最新成功构建
curl -s "$JENKINS_URL/job/$JOB_NAME/lastSuccessfulBuild/api/json" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | jq '{number, result}'

# 最新已完成构建（无论成功失败）
curl -s "$JENKINS_URL/job/$JOB_NAME/lastCompletedBuild/api/json" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | jq '{number, result}'

# 队列信息（如果构建正在排队）
curl -s "$JENKINS_URL/queue/api/json" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | jq '.items[] | {job: .task.name, queueId, why}'
```

### 3. 轮询构建直到完成

```bash
# 每 10 秒轮询一次，直到构建结束
JOB_NAME="my-pipeline"
JENKINS_URL="http://jenkins.example.com:8080"

build_number=$(curl -s "$JENKINS_URL/job/$JOB_NAME/lastBuild/buildNumber" \
  -u "$JENKINS_USER:$JENKINS_TOKEN")
[[ -z "$build_number" ]] && echo "未找到构建" && exit 1

while true; do
  result=$(curl -s "$JENKINS_URL/job/$JOB_NAME/$build_number/api/json" \
    -u "$JENKINS_USER:$JENKINS_TOKEN" | jq -r '.result')
  echo "[$(date '+%H:%M:%S')] 构建 #$build_number 结果: $result"
  case "$result" in
    null) sleep 10 ;;
    SUCCESS|FAILURE|ABORTED|UNSTABLE) break ;;
    *) echo "未知结果: $result"; break ;;
  esac
done
echo "最终结果: $result"
```

### 4. 下载制品

```bash
# 列出构建制品
curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/api/json" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | jq '.artifacts[].relativePath'

# 下载指定制品
ARTIFACT_PATH="path/to/artifact.jar"
curl -s -o /tmp/artifact.jar \
  "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/artifact/$ARTIFACT_PATH" \
  -u "$JENKINS_USER:$JENKINS_TOKEN"

# 下载全部制品为 ZIP
# 注意：Jenkins 可能会重定向，需要加 -L
curl -s -L -o /tmp/build-$BUILD_NUMBER.zip \
  "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/artifact/*zip*/**" \
  -u "$JENKINS_USER:$JENKINS_TOKEN"
```

### 5. 获取控制台输出

```bash
# 完整控制台日志（纯文本）
curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/consoleText" \
  -u "$JENKINS_USER:$JENKINS_TOKEN"

# 最后 50 行（适合轮询时查看进度）
curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/logText/progressiveText" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | tail -50

# JSON 结构化输出（含增量偏移，适合流式读取）
curl -s "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUMBER/logText/progressiveText/api/json" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | jq '{moreContent, done, contentSize}'
```

### 6. Jenkins CLI（高级操作）

```bash
# 下载 jenkins-cli.jar（一次性）
JENKINS_CLI_JAR=/tmp/jenkins-cli.jar
if [[ ! -f "$JENKINS_CLI_JAR" ]]; then
  curl -s -o "$JENKINS_CLI_JAR" \
    "$JENKINS_URL/jnlpJars/jenkins-cli.jar"
fi

# 执行 Jenkins CLI 命令
java -jar "$JENKINS_CLI_JAR" \
  -s "$JENKINS_URL" \
  -auth "$JENKINS_USER:$JENKINS_TOKEN" \
  list-jobs

# 通过 CLI 触发构建
java -jar "$JENKINS_CLI_JAR" \
  -s "$JENKINS_URL" \
  -auth "$JENKINS_USER:$JENKINS_TOKEN" \
  build "JOB_NAME" \
  -p PARAM1=value1 \
  -p PARAM2=value2

# 在任务之间复制制品
java -jar "$JENKINS_CLI_JAR" \
  -s "$JENKINS_URL" \
  -auth "$JENKINS_USER:$JENKINS_TOKEN" \
  copy-artifacts "SOURCE_JOB" \
  --include-objects "*.jar" \
  --dest "TARGET_JOB"
```

## Python 脚本模式

适用于复杂工作流（如解析测试结果、对比制品）：

```python
import requests
import time
import os

JENKINS_URL = os.environ["JENKINS_URL"]
JENKINS_USER = os.environ["JENKINS_USER"]
JENKINS_TOKEN = os.environ["JENKINS_TOKEN"]
auth = (JENKINS_USER, JENKINS_TOKEN)

def trigger_build(job_name, parameters=None):
    """触发 Jenkins 构建，支持参数化构建。"""
    url = f"{JENKINS_URL}/job/{job_name}/buildWithParameters"
    if parameters:
        resp = requests.post(url, data=parameters, auth=auth, timeout=30)
    else:
        resp = requests.post(url, auth=auth, timeout=30)
    resp.raise_for_status()
    # 返回 201 Created 表示成功加入队列
    return resp.status_code

def get_build_info(job_name, build_number="lastBuild"):
    """获取构建元数据（字典格式）。"""
    url = f"{JENKINS_URL}/job/{job_name}/{build_number}/api/json"
    resp = requests.get(url, auth=auth, timeout=30)
    resp.raise_for_status()
    return resp.json()

def poll_build(job_name, build_number=None, interval=10, timeout=3600):
    """轮询构建直到完成。"""
    if build_number is None:
        build_number = get_build_info(job_name, "lastBuild")["number"]
    start = time.time()
    while time.time() - start < timeout:
        info = get_build_info(job_name, build_number)
        result = info.get("result")
        print(f"[{int(time.time()-start)}s] #{build_number}: {result}")
        if result is not None:
            return result
        time.sleep(interval)
    raise TimeoutError(f"构建 #{build_number} 在 {timeout} 秒内未完成")

def download_artifact(job_name, build_number, artifact_path, dest):
    """下载单个制品文件。"""
    url = f"{JENKINS_URL}/job/{job_name}/{build_number}/artifact/{artifact_path}"
    resp = requests.get(url, auth=auth, timeout=60, stream=True, allow_redirects=True)
    resp.raise_for_status()
    with open(dest, "wb") as f:
        for chunk in resp.iter_content(chunk_size=8192):
            f.write(chunk)
    return dest

# 示例用法
trigger_build("my-maven-pipeline", {"BRANCH": "develop", "VERSION": "1.2.3"})
final_result = poll_build("my-maven-pipeline")
print(f"构建结果: {final_result}")
```

## 通过 Jenkins 调用 Maven / Gradle 打包

很多团队将 Maven/Gradle 调用封装在 Jenkins 内部：

```bash
# 触发 Maven 构建并指定 goals
curl -X POST "$JENKINS_URL/job/maven-build/buildWithParameters" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" \
  -d "MAVEN_GOALS=clean package" \
  -d "MAVEN_PROFILES=prod" \
  -d "SKIP_TESTS=false"

# 触发 Gradle 构建
curl -X POST "$JENKINS_URL/job/gradle-build/buildWithParameters" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" \
  -d "GRADLE_TASK=bootJar" \
  -d "JAVA_OPTS=-Xmx2g"
```

## 常见陷阱

1. **CRUMB 导致 403**：Jenkins 有 CSRF 保护。如果返回 403，先获取 crumb：
   ```bash
   CRUMB=$(curl -s "$JENKINS_URL/crumbIssuer/api/json" \
     -u "$JENKINS_USER:$JENKINS_TOKEN" | jq -r '.crumbRequestField')
   CRUMB_VALUE=$(curl -s "$JENKINS_URL/crumbIssuer/api/json" \
     -u "$JENKINS_USER:$JENKINS_TOKEN" | jq -r '.crumb')
   # 之后请求带上：-H "$CRUMB: $CRUMB_VALUE"
   ```

2. **build 和 buildWithParameters 混淆**：参数化任务必须用 `buildWithParameters`，用 `build` 会返回 400。

3. **API Token 无效**：有些 Jenkins 从 `$JENKINS_HOME/users/<username>/config.xml` 读取 token；UI 显示的 token 通常是 base64 编码后的，如 401 可尝试重新生成。

4. **制品路径编码**：从 `artifacts[].relativePath` 返回的路径已 URL 编码，下载前需要解码：
   ```bash
   ARTIFACT_PATH=$(echo "$REL_PATH" | jq -r '@uri')
   ```

5. **制品下载被重定向**：Jenkins 可能返回 302 重定向到实际文件地址，使用 `curl -L` 或 `requests.get(..., allow_redirects=True)`。

6. **Jenkins 在内网但通过网关访问**：API 响应中返回的 URL 可能指向内网地址。全程使用统一的网关地址作为 `JENKINS_URL`。

7. **构建进入队列但未开始**：返回 201 表示已加入队列，不代表已开始执行。可通过 `queue/item/{id}/api/json` 查看是否还在排队。

8. **所有命令工具均不可用**（terminal/execute_code/Docker 均挂）：触发构建后自己从 Response Header `Location` 提取 queue item URL，轮询该 URL 的 `/api/json` 直到 `executable.number` 出现，全程无需终端。如果连 `browser_navigate` 也超时（60秒），说明环境与 Jenkins 网络隔离，只能让用户在本地终端手动执行触发命令，不要反复重试 browser_navigate。

9. **构建名称/参数含中文/Unicode**：Jenkins API 返回 JSON 时会自动处理 Unicode 转义，`jq -r` 或 `resp.json()` 可直接解码。

### orange-aliyun 单独打包

适用于只需构建 orange-aliyun，不需要 orange-patch 的场景。

**任务路径**：`job/web/job/orange-aliyun`

**必填参数**：
| 参数 | 值 |
|------|-----|
| `build_type` | `vOrange` |
| `version_file` | 上传版本文件 |
| `options` | `update_code,npm_build,package,orange_patch` |

```bash
rm -f /tmp/jenkinsCookies.txt
curl -s -X POST "$JENKINS_URL/j_spring_security_check" -d "j_username=develop&j_password=dev1234&from=%2F" -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -L > /dev/null
CRUMB=$(curl -s "$JENKINS_URL/crumbIssuer/api/json" -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -u "develop:dev1234" | python3 -c "import sys,json; print(json.load(sys.stdin).get('crumb',''))")
curl -s -X POST "$JENKINS_URL/job/web/job/orange-aliyun/buildWithParameters" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" -H "Jenkins-Crumb: $CRUMB" \
  -F "build_type=vOrange" \
  -F "options=update_code,npm_build,package,orange_patch" \
  -F "version_file=@/tmp/version_file.txt"
```

---

### orange-patch 补丁包构建（独立使用）

适用于已有 vOrange 包，只需生成补丁包的场景。

**任务路径**：`job/web/job/orange-patch`

**必填参数**：
| 参数 | 值 |
|------|-----|
| `orange_package` | `orange-patch-YYYYMMDDHHMMSS.zip` |
| `orange_module` | **必须从 orange-aliyun 构建日志中提取**，不要询问用户。日志中会打印 "orange-patch-YYYYMMDDHHMMSS.zip"，从中解析日期时间填入 orange_package，orange_module 从 version_file 中对应的项目名匹配（如 version_file 中有 `pex`，则 orange_module=pex）。 |
```

### 构建确认模板

向用户展示时使用以下格式：

```
**Job**: orange-aliyun

**Build Parameters**:
| 参数 | 值 |
|------|-----|
| build_type | vOrange |
| options | update_code,npm_build,package,orange_patch |
| version_file | /tmp/version_file.txt (N条记录) |

**确认执行？** 请回复确认后我立即触发构建。
```

### ⚠️ 重要：获取构建号（不要等用户告知）

**触发后必须自己拿到构建号**，不允许让用户告知。

触发 `buildWithParameters` 成功后，Jenkins 返回 201 并在 Response Header 中包含 `Location: .../queue/item/{id}/`，随后该 queue item 的 API 响应中包含 `executable.number`。

**用 Python/requests 的标准流程：**
```python
import requests, time

resp = session.post(f"{JENKINS_URL}/job/{JOB_PATH}/buildWithParameters",
    files=build_params, auth=auth, headers=headers, timeout=60)

# Jenkins 201 = 已入队，Location header 指向 queue item
location = resp.headers.get('Location', resp.headers.get('location', ''))
print(f"Queue location: {location}")

# 轮询 queue item 直到有 executable.number
if location:
    for _ in range(30):
        q_resp = session.get(location + '/api/json', auth=auth, timeout=30)
        q_data = q_resp.json()
        if q_data.get('executable'):
            build_number = q_data['executable']['number']
            print(f"Build number: {build_number}")
            break
        time.sleep(2)
```

**不要问用户构建号**，触发后自己解析。如果所有命令工具都不可用（terminal/execute_code 均无），用 `browser_navigate` 访问 Jenkins 页面或告知用户在他们的终端手动触发。

### 监控构建状态

**⚠️ 重要：用户指定了 build 号后，监控该 build 号，不要用 lastBuild**

- `lastBuild` 会随着队列推进而变化，可能指向别人的构建
- 每次轮询都要用用户提供的 build 号（如 #30191）
- 直接请求 `$JOB_PATH/{build_number}/api/json`，不依赖 `lastBuild`

```bash
# 错误示范：lastBuild 可能是别人正在运行的构建
curl -s "$JENKINS_URL/$JOB_PATH/lastBuild/api/json" ...

# 正确示范：使用用户指定的 build 号
curl -s "$JENKINS_URL/$JOB_PATH/30191/api/json" ...
```

### orange-aliyun 前端打包 → orange-patch 完整流程（已验证）

**适用场景**：需要完整构建前端包 + 生成补丁包时使用。

**总流程**：
1. orange-aliyun 打包（vOrange）→ 产出 `vOrange_YYYYMMDDHHMMSS` + `orange_YYYYMMDDHHMMSS.zip`
2. orange-patch 构建补丁包（需用户确认 orange_module）

---

## Step 1：触发 orange-aliyun 打包

**任务路径**：`job/web/job/orange-aliyun`

### 必填参数

| 参数名 | 值 | 说明 |
|--------|-----|------|
| `build_type` | `vOrange` | 构建类型（固定值） |
| `version_file` | 上传文件 | **必须上传版本文件** |
| `options` | `update_code,npm_build,package,orange_patch` | 构建选项 |

### version_file 格式（3列，空格分隔）
```
日期时间 项目名 分支名
```
示例：
```
20250825172125 orange Release_20250830-5eccbc93
20260304104815 home Feature_20260228_tianJingDaXueZhiHuiCaiWu-b1449c99
20260328165400 pex Feature_20250430_tianJin-HEAD
```

### 触发命令
```bash
# 1. 重新认证
rm -f /tmp/jenkinsCookies.txt
curl -s -X POST "$JENKINS_URL/j_spring_security_check" \
  -d "j_username=develop&j_password=dev1234&from=%2F" \
  -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -L > /dev/null

CRUMB=$(curl -s "$JENKINS_URL/crumbIssuer/api/json" \
  -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" | python3 -c "import sys,json; print(json.load(sys.stdin).get('crumb',''))")

# 2. 生成 version_file（根据实际项目修改）
cat > /tmp/version_file.txt << 'EOF'
20250825172125 orange Release_20250830-5eccbc93
20260304104815 home Feature_20260228_tianJingDaXueZhiHuiCaiWu-b1449c99
20260328165400 pex Feature_20250430_tianJin-HEAD
EOF

# 3. 触发构建
curl -s -X POST "$JENKINS_URL/job/web/job/orange-aliyun/buildWithParameters" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" \
  -H "Jenkins-Crumb: $CRUMB" \
  -F "build_type=vOrange" \
  -F "options=update_code,npm_build,package,orange_patch" \
  -F "version_file=@/tmp/version_file.txt"

# 4. 获取构建号（lastBuild 不一定是你的，等队列）
sleep 3
curl -s "$JENKINS_URL/job/web/job/orange-aliyun/lastBuild/buildNumber" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt -u "develop:dev1234"
```

### 监控构建
```bash
# 轮询直到完成
BUILD_NUM=30191  # 替换为实际构建号
for i in $(seq 1 60); do
  RESULT=$(curl -s "$JENKINS_URL/job/web/job/orange-aliyun/$BUILD_NUM/api/json" \
    -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
    -u "develop:dev1234" | python3 -c "import sys,json; print(json.load(sys.stdin).get('result','RUNNING'))" 2>/dev/null)
  echo "[$i] #$BUILD_NUM: $RESULT"
  if echo "$RESULT" | grep -qE "SUCCESS|FAILURE|ABORTED|UNSTABLE"; then
    break
  fi
  sleep 15
done
```

### 产物获取
```bash
# 从构建日志提取产物信息
curl -s "$JENKINS_URL/job/web/job/orange-aliyun/$BUILD_NUM/consoleText" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt -u "develop:dev1234" | grep -E "vOrange_|orange_.*zip|外网下载地址"

# 典型输出：
# vOrange_20260506150807
# 外网下载地址：http://223.223.178.68:2004/orange-file/vOrange_20260506150807
# orange_20260506150807.zip
# 外网下载地址：http://223.223.178.68:2004/orange-file/orange_20260506150807.zip
```

---

## Step 2：触发 orange-patch 构建补丁包

**任务路径**：`job/web/job/orange-patch`

### 必填参数

| 参数名 | 值 | 说明 |
|--------|-----|------|
| `orange_package` | `orange-patch-YYYYMMDDHHMMSS.zip` | 从Step1日志获取 |
| `orange_module` | **必须询问用户** | 模块名，如 pex/home/orange 等 |

### orange_module 可选值（需询问用户）
- `pex`
- `home`
- `orange`
- 等（根据实际项目填写）

### 触发命令
```bash
# 1. 重新认证（cookie可能已过期）
rm -f /tmp/jenkinsCookies.txt
curl -s -X POST "$JENKINS_URL/j_spring_security_check" \
  -d "j_username=develop&j_password=dev1234&from=%2F" \
  -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -L > /dev/null

CRUMB=$(curl -s "$JENKINS_URL/crumbIssuer/api/json" \
  -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" | python3 -c "import sys,json; print(json.load(sys.stdin).get('crumb',''))")

# 2. 触发构建（orange_module 必须先询问用户确认）
ORANGE_PKG="orange-patch-YYYYMMDDHHMMSS.zip"  # 从Step1替换
ORANGE_MOD="pex"  # 询问用户确认

curl -s -X POST "$JENKINS_URL/job/web/job/orange-patch/buildWithParameters" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" \
  -H "Jenkins-Crumb: $CRUMB" \
  -F "orange_package=$ORANGE_PKG" \
  -F "orange_module=$ORANGE_MOD"
```

### 产物获取
```bash
# 从构建日志提取产物
curl -s "$JENKINS_URL/job/web/job/orange-patch/$BUILD_NUM/consoleText" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt -u "develop:dev1234" | grep -E "update_patch_|vOrange_|外网:"

# 典型输出：
# 文件名: update_patch_pex_20260506151923.zip
# 外网: http://223.223.178.68:2004/jenkins-orange-patch/update_patch_pex_20260506151923.zip
# vOrange_pex_20260506151923
# 外网: http://223.223.178.68:2004/jenkins-orange-patch/vOrange_pex_20260506151923
```

---

## 完整执行示例

```bash
# Step 1: 生成 version_file（根据实际项目）
cat > /tmp/version_file.txt << 'EOF'
20250825172125 orange Release_20250830-5eccbc93
20260304104815 home Feature_20260228_tianJingDaXueZhiHuiCaiWu-b1449c99
20260328165400 pex Feature_20250430_tianJin-HEAD
EOF

# Step 2: 认证 + 触发 orange-aliyun
rm -f /tmp/jenkinsCookies.txt
curl -s -X POST "$JENKINS_URL/j_spring_security_check" -d "j_username=develop&j_password=dev1234&from=%2F" -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -L > /dev/null
CRUMB=$(curl -s "$JENKINS_URL/crumbIssuer/api/json" -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -u "develop:dev1234" | python3 -c "import sys,json; print(json.load(sys.stdin).get('crumb',''))")

curl -s -X POST "$JENKINS_URL/job/web/job/orange-aliyun/buildWithParameters" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" -H "Jenkins-Crumb: $CRUMB" \
  -F "build_type=vOrange" \
  -F "options=update_code,npm_build,package,orange_patch" \
  -F "version_file=@/tmp/version_file.txt"

# Step 3: 等待 orange-aliyun 完成，从日志获取 orange-patch-YYYYMMDDHHMMSS.zip
# （用户确认 orange_module 后继续）

# Step 4: 认证 + 触发 orange-patch
rm -f /tmp/jenkinsCookies.txt
curl -s -X POST "$JENKINS_URL/j_spring_security_check" -d "j_username=develop&j_password=dev1234&from=%2F" -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -L > /dev/null
CRUMB=$(curl -s "$JENKINS_URL/crumbIssuer/api/json" -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -u "develop:dev1234" | python3 -c "import sys,json; print(json.load(sys.stdin).get('crumb',''))")

curl -s -X POST "$JENKINS_URL/job/web/job/orange-patch/buildWithParameters" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" -H "Jenkins-Crumb: $CRUMB" \
  -F "orange_package=orange-patch-YYYYMMDDHHMMSS.zip" \
  -F "orange_module=pex"  # 替换为实际模块名
```

- **Terminal 命令被拦截（BLOCKED）**：Jenkins 认证 + 构建触发不要写成一行多命令链式（`rm -f ... && curl ...`），拆成多条独立 `terminal` 调用，每条单独审批
- **监控到别人的构建**：用户说"我的构建是 #XXX"后，必须用这个 build 号直接查询，不要依赖 `lastBuild`

## 参考资料

- [orange-patch 构建日志提取模式](./references/orange-patch-console-patterns.md) — 从 orange-aliyun 控制台提取 `orange-patch-YYYYMMDDHHMMSS.zip` 的正则模式，以及两阶段完整流程记录

## 验证清单

- [ ] Jenkins URL 可达（网络/防火墙正常）
- [ ] API Token 有效（用 `/api/json` 端点测试）
- [ ] 任务名称存在（通过 `.../api/json` 列出任务验证）
- [ ] 构建触发成功（返回 201 或队列位置）
- [ ] 构建以预期结果结束（SUCCESS/FAILURE/ABORTED/UNSTABLE）
- [ ] 制品已下载且文件大小 > 0
- [ ] 控制台日志显示预期构建步骤

## 典型场景

### 完整流程：触发 → 等待 → 下载 JAR

```bash
JOB_NAME="backend-service-pipeline"
ARTIFACT="backend/target/backend-1.0.0.jar"
AUTH="$JENKINS_USER:$JENKINS_TOKEN"
JENKINS_URL="http://jenkins.example.com:8080"

# 1. 触发构建
echo "正在触发构建..."
curl -s -X POST \
  "$JENKINS_URL/job/$JOB_NAME/buildWithParameters" \
  -u "$AUTH" \
  -d "VERSION=1.0.0" \
  -d "ENV=staging"
echo "触发完成"

# 2. 等待构建完成（最多 30 分钟）
echo "轮询构建状态..."
for i in $(seq 1 180); do
  result=$(curl -s "$JENKINS_URL/job/$JOB_NAME/lastBuild/api/json" \
    -u "$AUTH" | jq -r '.result')
  echo "[$i] result=$result"
  case "$result" in
    SUCCESS) echo "构建成功"; break ;;
    FAILURE|ABORTED|UNSTABLE) echo "构建失败: $result"; exit 1 ;;
  esac
  sleep 10
done

# 3. 下载制品
BUILD_NUM=$(curl -s "$JENKINS_URL/job/$JOB_NAME/lastBuild/api/json" \
  -u "$AUTH" | jq -r '.number')
echo "从构建 #$BUILD_NUM 下载制品..."
curl -s -L -o /tmp/backend.jar \
  "$JENKINS_URL/job/$JOB_NAME/$BUILD_NUM/artifact/$ARTIFACT" \
  -u "$AUTH"
ls -lh /tmp/backend.jar
```
