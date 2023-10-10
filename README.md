# jenkins-study

클라우드 환경에서 젠킨스 다운로드

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

    
