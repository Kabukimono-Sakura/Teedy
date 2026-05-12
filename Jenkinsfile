pipeline {
    agent any

    environment {
        // 定义环境变量 - Jenkins 凭据配置
        // 需提前在 Jenkins 凭据管理中创建 ID 为 'dockerhub_credentials' 的凭据
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub_credentials')
        
        // Docker Hub 仓库名称（请将 xx 修改为您真实的用户名）
        DOCKER_IMAGE = 'xx/teedy-app' 
        
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
                    userRemoteConfigs: [[url: 'https://github.com/xx/Teedy.git']]
                )
                // 执行 Maven 打包，跳过测试
                sh 'mvn -B -DskipTests clean package'
            }
        }

        // 第二阶段：构建 Docker 镜像
        stage('Building image') {
            steps {
                script {
                    // 假设 Dockerfile 位于代码根目录
                    docker.build("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}")
                }
            }
        }

        // 第三阶段：将镜像上传到 Docker Hub
        stage('Upload image') {
            steps {
                script {
                    // 使用 registry 地址和凭据登录并推送
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub_credentials') {
                        // 推送带构建编号的版本
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push()
                        
                        // 同时推送一个名为 latest 的标签（可选）
                        docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        // 第四阶段：在当前服务器上运行容器
        stage('Run containers') {
            steps {
                script {
                    // 停止并删除已存在的同名容器，防止端口冲突（|| true 确保即使没有旧容器也不会报错）
                    sh 'docker stop teedy-container-8081 || true'
                    sh 'docker rm teedy-container-8081 || true'
                    
                    // 运行新容器，映射宿主机 8081 到容器 8080
                    docker.image("${env.DOCKER_IMAGE}:${env.DOCKER_TAG}").run(
                        '--name teedy-container-8081 -d -p 8081:8080'
                    )
                    
                    // （可选）查看容器运行状态
                    sh 'docker ps --filter "name=teedy-container"'
                }
            }
        }
    }
}