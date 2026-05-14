pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'M3'
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
        stage('Maven Build'){
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
        }
        // SSH를 이용한 배포
        stage('SSH Publish') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'target',
                transfers: [sshTransfer(cleanRemote: false, excludes: '', 
                execCommand: '''fuser -k 8080/tcp
                export BUILD_ID=Spring-PetClinic                
                 nohup java -jar /home/ubuntu/spring-petclinic-4.0.0-SNAPSHOT.jar >> nohup.out 2>&1 &''',
                execTimeout: 120000, 
                flatten: false,
                makeEmptyDirs: false, 
                noDefaultExcludes: false, patternSeparator: '[, ]+',
                remoteDirectory: '',
                remoteDirectorySDF: false, removePrefix: 'target', 
                sourceFiles: 'target/*.jar')],
                usePromotionTimestamp: false,
                useWorkspaceInPromotion: false,
                verbose: false)])
            }
        }
        
    }   
}
