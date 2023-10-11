# jenkins-study

클라우드 환경(Google Cloud Platform)에서 젠킨스 다운로드
1. Jenkins서버를 위한 8080 포트 모든 경로로 개방
2. test를 위한 api 서버를 위한 8081 포트 모든 경로로 개방
3. ssh을 통해 클라우드 인스턴스 접속

ssh을 통해 클라우드 인스턴스 접속

    ssh -i {ssh private key} {passphrase for key}@{외부 ip} 

Jenkins GPG Key 다운로드 및 시스템에 추가

    curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null

Jenkins APT 저장소 엔트리 추가

    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null

자바 17버전, Jenkins 다운로드

    sudo apt-get update
    sudo apt-get install fontconfig openjdk-17-jre
    sudo apt-get install jenkins

자바 버전 확인, Jenkins 상태 확인

    java --version
    sudo systemctl status jenkins

Jenkins 서버 접속 후 파이브라인 작성

    http://{ip}:8080

Jenkins 파이프라인 작성
1. git webhook 등록 - 해당 레포지토리에 push 이벤트 발생 시 Jenkins 파이브라인 실행
2. Jenkins 콘솔에서 새아이템 생성 후 Build Triggers - GitHub hook trigger for GITScm polling 체크
3. 파이브라인 작성

Pipeline script

    pipeline {
        agent any
    
        stages {
            stage('Cloning') {
                steps {
                    git url: 'https://github.com/Sudongk/api-for-gcp.git',
                        branch: 'main'
                }
            }
            
            stage('Build') {
                steps {
                    sh ('chmod 744 gradlew')
                    sh './gradlew build'
                }
    
                post {
                    success {
                        echo 'success'
                    }
                    
                    failure {
                        echo 'failure'
                    }
                }
            }
            
            stage('mv') {
                steps {
                    sh 'sudo mv build/libs/*T.jar /usr/share/app.jar'   
                }
            }
            
            stage('start') {
                steps {
                    sh 'sudo sh /usr/share/start.sh'
                }
            }
        }
    }

Jenkins 플러그인 Publish over SSH을 이용한 gitHub Webhook을 이용한 Jenkins 빌드 서버에서 빌드 후 jar 파일만 배포 서버로 전송

Pipeline script

    pipeline {
        agent any
    
        stages {
            stage('Cloning') {
                steps {
                    git url: 'https://github.com/Sudongk/api-for-gcp.git',
                        branch: 'main'
                }
            }
            
            stage('Build') {
                steps {
                    sh ('chmod 744 gradlew')
                    sh './gradlew build'
                }
    
                post {
                    success {
                        echo 'success'
                    }
                    
                    failure {
                        echo 'failure'
                    }
                }
            }

            // Jenkins 플러그인 Publish over SSH을 이용한 gitHub Webhook을 이용한 Jenkins 빌드 서버에서 빌드 후 jar 파일만 배포 서버로 전송
            stage('Deploy') {
                steps {
                    sshPublisher(
                        publishers : [
                            sshPublisherDesc(
                                configName: 'deploy',
                                transfers : [
                                    sshTransfer(
                                        sourceFiles: 'build/libs/*T.jar',
                                        remoteDirectory: '',
                                        removePrefix:'build/libs',
                                        execCommand: 'java -jar *.jar > app.out 2>&1 &'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }

start.sh 스크립트 작성

    kill -9 $(ps -ef | grep "java -jar" | grep -v grep | awk '{print $2}')
    java -jar /usr/share/app.jar &

