# Lab 1：第一个 CI/CD Pipeline 实验流程

## 1. 实验目标

本实验的目标是搭建一条基础 CI/CD 流水线，使一个 Web 应用可以完成以下流程：

1. 从 Git 仓库获取源代码。
2. 自动构建项目。
3. 自动执行测试。
4. 自动部署到 Linux 虚拟机。
5. 从宿主机浏览器访问部署后的 Web 应用。
6. 通过网络配置实现远程触发流水线。

最终需要证明：代码提交后，流水线可以完成 build、test、deploy，并且应用可以被访问。

## 2. 实验准备

### 2.1 团队工具

需要先准备团队协作工具：

- 创建 GitHub 项目仓库。
- 明确团队成员分工。
- 准备实验记录文档，用于保存命令、配置、问题和解决方法。
- 定期提交实验进度到 GitHub，避免虚拟机故障导致成果丢失。

### 2.2 虚拟机环境

需要准备一台 Linux 虚拟机，作为 CI/CD 实验环境。

建议安装或配置：

- Linux 虚拟机。
- Git。
- GitLab。
- Jenkins。
- Java、Maven、Node.js、npm 或其他 Web 项目需要的运行环境。
- SSH 服务。
- 必要的端口映射。

注意：实验要求自己搜索并验证安装方法。不要直接复制过时教程中的命令，每个命令应先手动运行确认有效，再写入自动化脚本或流水线。

## 3. 选择 Web 应用

需要选择一个适合 CI/CD 实验的 Web 应用项目。

项目应满足：

- 有完整源代码。
- 有清楚的构建方式。
- 有可执行测试。
- 可以在 Linux 虚拟机中运行。
- 可以通过 HTTP 从浏览器访问。

可选技术栈示例：

- Java Spring Boot + Maven。
- Node.js + Express。
- Python Flask 或 Django。

建议选择结构简单、依赖少、测试命令明确的项目，避免把时间花在复杂业务逻辑或环境问题上。

## 4. 本地手动验证

在配置流水线之前，必须先在虚拟机中手动验证项目可以运行。

需要完成：

1. 克隆项目代码。
2. 安装依赖。
3. 执行构建命令。
4. 执行测试命令。
5. 启动 Web 应用。
6. 在浏览器中访问应用。

示例流程：

```bash
git clone <repository-url>
cd <project-directory>
<install-dependencies-command>
<build-command>
<test-command>
<run-command>
```

只有当这些命令全部手动验证成功后，才应该进入 CI/CD pipeline 配置。

## 5. 配置 Git 仓库

需要将 Web 应用代码放入 GitHub 或 GitLab 仓库。

推荐流程：

1. 创建远程仓库。
2. 将项目代码提交到仓库。
3. 添加必要的 CI/CD 配置文件。
4. 提交 README、实验记录和配置说明。

常用命令：

```bash
git status
git add .
git commit -m "Initial CI/CD lab setup"
git push
```

## 6. 配置 Jenkins Pipeline

Jenkins Pipeline 需要完成以下阶段：

1. Checkout：拉取代码。
2. Build：构建项目。
3. Test：执行测试。
4. Deploy：部署应用到虚拟机。
5. Verify：验证应用是否可访问。

示例 Jenkinsfile 结构：

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git '<repository-url>'
            }
        }

        stage('Build') {
            steps {
                sh '<build-command>'
            }
        }

        stage('Test') {
            steps {
                sh '<test-command>'
            }
        }

        stage('Deploy') {
            steps {
                sh '<deploy-command>'
            }
        }

        stage('Verify') {
            steps {
                sh 'curl -f http://localhost:<app-port>'
            }
        }
    }
}
```

需要根据实际项目替换：

- `<repository-url>`
- `<build-command>`
- `<test-command>`
- `<deploy-command>`
- `<app-port>`

## 7. 部署 Web 应用

部署阶段的目标是让应用在虚拟机中稳定运行，并可以从宿主机访问。

可以采用以下方式之一：

- 直接用命令启动应用。
- 使用后台进程运行应用。
- 使用 systemd 管理服务。
- 使用 Docker 运行应用。

基础要求：

- 应用监听固定端口。
- Jenkins 构建完成后可以启动或重启应用。
- 应用启动后可以通过 `curl` 验证。
- 宿主机浏览器可以访问应用页面。

示例验证命令：

```bash
curl -f http://localhost:<app-port>
```

## 8. 网络配置

实验还要求改进虚拟机与宿主机之间的网络配置。

需要完成：

### 8.1 共享目录

配置 Windows Host 与 Linux VM 之间的共享目录，用于交换文件、截图、配置或实验记录。

### 8.2 端口转发

需要配置宿主机到虚拟机的端口转发。

常见端口：

- SSH：用于远程连接虚拟机。
- HTTP：用于访问 Web 应用。
- Jenkins：用于访问 Jenkins 页面。
- GitLab：用于访问 GitLab 页面。

示例：

| 服务 | 虚拟机端口 | 宿主机端口 |
| --- | --- | --- |
| SSH | 22 | 2222 |
| Web App | 8080 | 8080 |
| Jenkins | 8081 | 8081 |
| GitLab | 80 | 8082 |

实际端口可以根据本机环境调整，避免端口冲突。

### 8.3 SSH Client

在宿主机配置 SSH client，确认可以连接虚拟机。

示例：

```bash
ssh user@localhost -p 2222
```

### 8.4 从宿主机触发 Pipeline

需要能从 Windows Host 使用 `curl` 触发 Jenkins pipeline job。

示例：

```bash
curl -X POST http://localhost:<jenkins-port>/job/<job-name>/build
```

如果 Jenkins 启用了认证或 CSRF crumb，需要额外配置 token 或 crumb。

### 8.5 从第二台虚拟机触发 Pipeline

进一步要求：从第二台 VM 中使用 `curl` 触发第一台 VM 上的 pipeline。

需要确认：

- 两台虚拟机网络互通。
- Jenkins 端口可以从第二台 VM 访问。
- 触发命令可以成功启动 job。

## 9. 进阶任务

以下任务不是强制要求，但可以为最终项目做准备：

1. 创建第二台 VM，作为 CI/CD slave。
2. 创建第三台 VM，作为最终部署环境。
3. 修改 pipeline：
   - 在 slave 上执行 build 和 test。
   - 将应用部署到最终 VM。
4. 使用 SSH、scp、rsync 或 Docker 完成远程部署。

## 10. 验收标准

实验完成时，应能展示以下结果：

- GitHub 或 GitLab 中有项目代码。
- Jenkins 或 GitLab CI 中有可运行的 pipeline。
- Pipeline 至少包含 build、test、deploy 阶段。
- 测试阶段可以成功执行。
- 部署后 Web 应用可以访问。
- 宿主机浏览器可以访问虚拟机中的 Web 应用。
- 可以使用 `curl` 从宿主机触发 pipeline。
- 实验过程中的命令、配置、问题和解决方案已记录。

## 11. 建议记录内容

实验过程中建议持续记录：

- 使用的操作系统版本。
- 虚拟机网络配置。
- GitLab 安装步骤。
- Jenkins 安装步骤。
- Web 应用来源。
- 构建命令。
- 测试命令。
- 部署命令。
- Jenkinsfile 内容。
- 端口转发配置。
- 遇到的问题和解决方法。
- 最终访问地址。

## 12. 推荐执行顺序

建议按照以下顺序完成实验：

1. 创建 GitHub 仓库和实验文档。
2. 安装 Linux 虚拟机基础环境。
3. 安装 Git、Jenkins、GitLab。
4. 选择一个简单 Web 应用。
5. 在虚拟机中手动 build、test、run。
6. 将项目提交到 Git 仓库。
7. 编写 Jenkinsfile。
8. 在 Jenkins 中创建 pipeline job。
9. 运行 pipeline 并修复错误。
10. 配置 Web 应用部署。
11. 配置端口转发，让宿主机可以访问应用。
12. 使用 `curl` 从宿主机触发 pipeline。
13. 记录最终结果并提交到 GitHub。

## 13. 关键注意事项

- 不要一开始就写 pipeline，先手动确认命令可行。
- 每一步成功后及时截图或记录命令输出。
- 遇到问题先看日志，再搜索错误信息。
- 保持 pipeline 简单，先完成基本 build、test、deploy。
- 不要把复杂项目作为第一个 CI/CD 实验对象。
- 定期备份虚拟机和 Git 仓库。
- 端口冲突时及时更换端口并更新文档。

