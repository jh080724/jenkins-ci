// 필요한 변수를 선언(젠킨스 환경변수를 끌고 올 수 있음.)
def dockerHubUsername="jh080724"
def dockerHubRepository="my-app-image"  // 도커 이미지 이름
def deployHost="172.31.10.141"  // 배포서버 private IP(같은 VPC 내 존재)

// Jenkins의 선언형 파이프라인 정의부 시작(Groovy 언어, 선언형)
pipeline {
    agent any   // 어느 Jenkins 서버에서도 실행이 가능

    stages {
        stage('Pull Codes from Github'){ // 스테이지 제목(user define)
            steps{
                checkout scm    // Jenkins와 연결된 소스컨트롤매니저(SCM: git 등)에서 코드를 가져오는 명령어
            }
        }
        stage('Build Codes by Gradle') {
            steps {
                sh """
                ./gradlew clean build
                """
            }
        }
        stage('Build Docker Image & Push to Docker Hub') {
            steps {
                sh """
                    # Docker Hub 로그인
                    echo "${env.DOCKER_HUB_PASSWORD}" | docker login -u "${dockerHubUsername}" --password-stdin

                    # Docker 이미지 빌드
                    docker build -t ${dockerHubUsername}/${dockerHubRepository}:${currentBuild.number} .

                    # Docker Hub로 푸시
                    docker push ${dockerHubUsername}/${dockerHubRepository}:${currentBuild.number}
                """
            }
        }
        stage('Deploy to AWS EC2 VM') {
            steps {
                sshagent(credentials: ["deploy-key"]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${deployHost} \
                     'docker login -u ${dockerHubUsername} -p ${env.DOCKER_HUB_PASSWORD}; \
                      docker pull ${dockerHubUsername}/${dockerHubRepository}:${currentBuild.number}; \
                      docker stop \$(docker ps -q) || true; \
                      docker run -d -p 80:8080 ${dockerHubUsername}/${dockerHubRepository}:${currentBuild.number};'
                    """
                }
            }
        }
    }
}