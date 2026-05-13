pipeline {
    agent any

    environment {
        // 定义环境变量 - Jenkins 凭据配置
        // 需提前在 Jenkins 凭据管理中创建 ID 为 'Docker-Hub' 的凭据
        DOCKER_HUB_CREDENTIALS = credentials('Docker-Hub')

        // Docker Hub 仓库名称
        DOCKER_IMAGE = 'kabukimonosakura/teedyjenkins'

        // 使用 Jenkins 的构建编号作为 Docker 镜像的 Tag
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        // 第一阶段：拉取代码并使用 Maven 编译
        stage('Build') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/Kabukimono-Sakura/Teedy.git']]
                )
                // 执行 Maven 打包，跳过测试
                sh 'mvn -B -DskipTests clean package'
            }
        }

        // 第二阶段：构建 Docker 镜像
        stage('Building image') {
            steps {
                // 使用 docker build 命令构建镜像（避免 Windows 路径过长问题）
                sh "docker build -t ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ."
            }
        }

        // 第三阶段：将镜像上传到 Docker Hub
        stage('Upload image') {
            steps {
                // 使用 docker login 直接登录（避免 Docker Pipeline 插件在 Windows 上的路径问题）
                sh "echo ${DOCKER_HUB_CREDENTIALS_PSW} | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin"
                // 推送带构建编号的版本
                sh "docker push ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                // 同时推送一个名为 latest 的标签
                sh "docker tag ${env.DOCKER_IMAGE}:${env.DOCKER_TAG} ${env.DOCKER_IMAGE}:latest"
                sh "docker push ${env.DOCKER_IMAGE}:latest"
            }
        }

        // 第四阶段：在当前服务器上运行三个容器
        stage('Run containers') {
            steps {
                // 停止并删除已存在的同名容器，防止端口冲突（|| true 确保即使没有旧容器也不会报错）
                sh 'docker stop teedy-container-8082 || true'
                sh 'docker rm teedy-container-8082 || true'
                sh 'docker stop teedy-container-8083 || true'
                sh 'docker rm teedy-container-8083 || true'
                sh 'docker stop teedy-container-8084 || true'
                sh 'docker rm teedy-container-8084 || true'

                // 运行三个容器，分别映射 8082、8083、8084 到容器 8080
                sh "docker run --name teedy-container-8082 -d -p 8082:8080 ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                sh "docker run --name teedy-container-8083 -d -p 8083:8080 ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                sh "docker run --name teedy-container-8084 -d -p 8084:8080 ${env.DOCKER_IMAGE}:${env.DOCKER_TAG}"

                // 查看容器运行状态
                sh 'docker ps --filter "name=teedy-container"'
            }
        }
    }
}
