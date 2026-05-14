pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'M3'
    }

    environment {
        DOCKERHUB_CRED = credentials('dockerCredential')
    }

    stages {
        // Github에서 소스코드 가져오기
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/hame2/spring-petclinic.git',
                branch: 'main'
            }
        }

        // Maven으로 Build
        stage('Maven Build') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }
        // Docker 이미지 생성
        stage('Docker Build') {
            steps {
                sh 'docker build -t spring-petclinic:${BUILD_NUMBER} .'
                sh 'docker tag spring-petclinic:${BUILD_NUMBER} hame2/spring-petclinic:latest'
            }
        }
        // Docker 이미지를 Docker Hub로 Push
        stage('Docker Push') {
            steps {
                sh 'echo ${DOCKERHUB_CRED_PSW} | docker login -u ${DOCKERHUB_CRED_USR} --password-stdin'
                sh 'docker push hame2/spring-petclinic:latest'
            }
        }
        // Docker 이미지 삭제
        stage('Docker Clean') {
            steps {
                sh '''
                docker rmi spring-petclinic:${BUILD_NUMBER}
                docker rmi hame2/spring-petclinic:latest
                '''
            }
        }
        // Docker Hub를 이용한 배포
        stage('Docker Deploy') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'target', 
                transfers: [sshTransfer(cleanRemote: false,  
                excludes: '',
                execCommand: '''
                docker rm -f $(docker ps -aq)
                docker rmi $(docker images -q)
                docker run -itd -p 80:8080 --name=spring-petclinic hame2/spring-petclinic:latest
                ''', 
                execTimeout: 120000, 
                flatten: false, 
                makeEmptyDirs: false, 
                noDefaultExcludes: false, 
                patternSeparator: '[, ]+', 
                remoteDirectory: '', 
                remoteDirectorySDF: false, 
                removePrefix: 'target', 
                sourceFiles: '')], 
                usePromotionTimestamp: false, 
                useWorkspaceInPromotion: false, 
                verbose: false)])
            }
        }
    }
}
