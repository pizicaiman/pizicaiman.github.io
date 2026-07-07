---
layout: book-article
title: "Jenkins Pipeline 高级用法"
excerpt: "Jenkins Shared Library、声明式流水线与并行阶段的高级实践。"
category_link: /books/devops/
category_title: DevOps 读书笔记
icon: ""
date: 2025-02-20
status: reading
tags: [Jenkins, Pipeline, CI/CD]
chapters:
  - title: "声明式 Pipeline 进阶"
    subtitle: "超越基础流水线的声明式语法"
    content: |
      ## 声明式 vs 脚本式

      Jenkins Pipeline 有两种语法：

      | 特性 | 声明式 (Declarative) | 脚本式 (Scripted) |
      |------|---------------------|-------------------|
      | 语法风格 | 结构化、预定义块 | Groovy 脚本，灵活 |
      | 学习曲线 | 低 | 高 |
      | 校验 | 启动前校验语法 | 运行时才发现错误 |
      | 适用场景 | 标准 CI/CD 流程 | 复杂动态逻辑 |

      ### 高级声明式特性

      ```groovy
      pipeline {
          agent { label 'linux && docker' }

          options {
              timeout(time: 1, unit: 'HOURS')
              retry(3)
              timestamps()
              buildDiscarder(logRotator(numToKeepStr: '20'))
          }

          environment {
              DOCKER_REGISTRY = credentials('docker-registry')
              DEPLOY_ENV = "${params.ENVIRONMENT}"
          }

          stages {
              stage('Build') {
                  when {
                      branch 'main'
                      expression { params.SKIP_BUILD != true }
                  }
                  steps {
                      sh 'mvn clean package'
                  }
              }

              stage('Deploy') {
                  parallel {
                      stage('US-East') {
                          steps { sh './deploy.sh us-east-1' }
                      }
                      stage('EU-West') {
                          steps { sh './deploy.sh eu-west-1' }
                      }
                  }
              }
          }

          post {
              always { junit '**/target/surefire-reports/*.xml' }
              success { slackSend color: 'good', message: "Build ${env.BUILD_NUMBER} succeeded" }
              failure { slackSend color: 'danger', message: "Build ${env.BUILD_NUMBER} failed" }
          }
      }
      ```

      > **关键技巧**：`when` 条件块支持 `branch`、`expression`、`changelog`、`tag` 等多种条件组合。

  - title: "Shared Library 架构设计"
    subtitle: "构建企业级 Pipeline 代码复用体系"
    content: |
      ## Shared Library 结构

      ```
      jenkins-shared-library/
      ├── src/                          # Groovy 源代码
      │   └── org/
      │       └── example/
      │           ├── DockerUtils.groovy
      │           └── K8sDeployer.groovy
      ├── vars/                         # 全局变量（Pipeline 步骤）
      │   ├── dockerBuild.groovy
      │   ├── k8sDeploy.groovy
      │   └── notifySlack.groovy
      ├── resources/                    # 静态资源（模板文件）
      │   └── templates/
      │       └── deployment.yaml.tpl
      └── Jenkinsfile                   # 库自身的测试流水线
      ```

      ### 全局变量定义

      ```groovy
      // vars/k8sDeploy.groovy
      def call(Map config = [:]) {
          def namespace = config.namespace ?: 'default'
          def image = config.image
          def replicas = config.replicas ?: 1

          kubernetesDeploy(
              configs: "k8s/deployment.yaml",
              enableConfigSubstitution: true,
              kubeconfigId: 'k8s-cluster'
          )
      }
      ```

      ### 在 Pipeline 中使用

      ```groovy
      @Library('jenkins-shared-library') _

      pipeline {
          stages {
              stage('Deploy') {
                  steps {
                      k8sDeploy(
                          namespace: 'production',
                          image: "myapp:${env.BUILD_NUMBER}",
                          replicas: 3
                      )
                  }
              }
          }
      }
      ```

      > **架构建议**：将通用逻辑封装为全局变量，业务特定逻辑放在项目级 Jenkinsfile 中。

  - title: "并行与分布式构建"
    subtitle: "加速流水线的并行执行策略"
    content: |
      ## 并行阶段

      ```groovy
      stage('Test') {
          parallel {
              stage('Unit Tests') {
                  steps { sh 'mvn test' }
              }
              stage('Integration Tests') {
                  steps { sh 'mvn verify -P integration' }
              }
              stage('Security Scan') {
                  steps { sh 'trivy image myapp:latest' }
              }
          }
      }
      ```

      ## 矩阵构建（Matrix）

      ```groovy
      stage('Cross-Browser Test') {
          matrix {
              axes {
                  axis {
                      name 'BROWSER'
                      values 'chrome', 'firefox', 'safari'
                  }
                  axis {
                      name 'PLATFORM'
                      values 'linux', 'macos'
                  }
              }
              stages {
                  stage('E2E Test') {
                      steps {
                          sh "npm run e2e -- --browser=${BROWSER} --platform=${PLATFORM}"
                      }
                  }
              }
          }
      }
      ```

      ## 性能优化建议

      | 优化项 | 方法 | 预期效果 |
      |--------|------|---------|
      | 并行测试 | `parallel` 块拆分测试套件 | 减少 50-70% 测试时间 |
      | 增量构建 | 基于 Git 变更的增量编译 | 减少 30-60% 构建时间 |
      | 缓存依赖 | `node_modules` / Maven 本地仓库缓存 | 减少 40-80% 依赖下载时间 |
      | 分布式 Agent | 多 Agent 并行执行不同阶段 | 提升整体吞吐量 |

      > **注意事项**：并行阶段共享同一 Workspace 时需注意文件锁和端口冲突问题。

---
