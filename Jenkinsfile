pipeline {

    // 全局环境变量
    environment {
        IMAGENAME     = 'webdemo'       // 镜像名称
        IMAGETAG      = '1.0.0'         // 镜像标签
        APPPORT       = '8089'          // 应用占用的端口
        APPDIR        = '/opt/app'      // 应用工作的目录
        WORKSPACE     = '/var/jenkins_home'
    }

    agent {
        docker {
            image 'mcr.microsoft.com/dotnet/sdk:6.0' 
            args '-v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker'            
        }
    }
    stages {

        
        // 开始构建，debug、test，在此过程中还原程序nuget依赖、输出 debug、单元测试等
        stage('Build') { 
            steps {
                echo 'building start! first,restore reference.'
                sh 'dotnet --version'
                sh 'dotnet restore'
            }
        }

        // 执行单元测试
        stage('Test') { 
            steps {
                sh 'dotnet test  --logger "console;verbosity=detailed"  --blame  --logger trx'
            }
        }
        
        // 正式发布
        stage('Publish') { 
            steps {
                sh 'dotnet publish src/WebDemo -c Release'
            }
        }

        // 部署应用，
        // 这里选择将应用打包为 docker 镜像
        stage('Deploy') { 

            steps {
                sh  'touch Dockerfile'
                sh  'env'
                sh  'echo "start edit Dockerfile"'
                sh  'echo "FROM mcr.microsoft.com/dotnet/aspnet:6.0" >> Dockerfile'
                sh  'echo "COPY src/WebDemo/bin/Release/netcoreapp6.0/publish ${APPDIR}" >> Dockerfile'
                sh  'echo "EXPOSE ${APPPORT}" >> Dockerfile'
                sh  'echo "WORKDIR ${APPDIR}" >> Dockerfile'
                sh  'echo \'ENTRYPOINT ["dotnet", "WebDemo.dll"]\' >> Dockerfile'

                sh 'cat Dockerfile'
                sh "docker build -t ${IMAGENAME}:${IMAGETAG} ."
            }
        }

        // 后续还可以执行 Docker 命令部署镜像，再使用健康检查等 API 检查容器是否正常，实现自动回退

    }
}
