#!groovy

pipeline {

    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: image-builder
  labels:
    robot: builder
spec:
  serviceAccount: jenkins-agent
  containers:
  - name: jnlp
  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.18.0-debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker/
        readOnly: true
  - name: kubectl
    image: bitnami/kubectl
    tty: true
    command:
    - cat
    securityContext:
      runAsUser: 1000
  - name: golang
    image: golang:1.21.3
    tty: true
    command:
    - cat
  volumes:
    - name: docker-config
      secret:
        secretName: credentials
        optional: false
"""
        }
    }

    environment {
        // Поміняйте APP_NAME та DOCKER_IMAGE_NAME на ваше імʼя та прізвище, відповідно.
        APP_NAME = 'your_app_name'
        DOCKER_IMAGE_NAME = 'your_docker_image_name'
    }

    stages {
        stage('Clone Repository') {
            steps {
                container(name: 'jnlp', shell: '/bin/bash') {
                    echo 'Pulling new changes'
                    // Крок клонування репозиторію
                    // TODO: ваш код з лабораторної № 4
                }
            }
        }
        stage('Compile') {
            steps {
                container(name: 'golang', shell: '/bin/bash') {
                    // Компіляція проекту на мові Go. Всі ці флаги необхідні для запуску на пустій файловій системі образу scratch :)
                    sh "CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GOFLAGS=-buildvcs=false go build -a -ldflags '-w -s -extldflags \"-static\"' -o ${APP_NAME} ."
                }
            }
        }

        stage('Unit Testing') {
            steps {
                container(name: 'golang', shell: '/bin/bash') {
                    echo 'Testing the application'
                    // Виконання юніт-тестів.
                    // TODO: ваш код з лабораторної № 4
                }
            }
        }
        stage('Build image') {
            // Не потрібно змінювати. Цей код працюватиме, якщо у вас правильний Dockerfile.
            environment {
                PATH = "/busybox:/kaniko:$PATH"
            }
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
                    /kaniko/executor --dockerfile="$(pwd)/Dockerfile" --context="dir:///$(pwd)" --build-arg "APP_NAME=${APP_NAME}" --destination ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                container(name: 'kubectl', shell: '/bin/bash') {
                    echo 'Deploying to Kubernetes'
                    // TODO: Потрібно зробити дві речі
                    // TODO: По-перше: якимось чином за допомогою bash підставте значення змінних DOCKER_IMAGE_NAME і BUILD_NUMBER у свій Deployment.
                    // TODO: Підказка: bitnami/kubectl має доступну утиліту 'sed'
                    // TODO: Але ви можете використовувати будь-яке інше рішення (Kustomize, тощо)
                    // TODO: По-друге: використовуйте kubectl apply з контейнера kubectl щоб застосувати маніфести з директорії k8s
                }
            }
        }
        stage('Test deployment') {
            agent {
                kubernetes {
                    yaml """
apiVersion: v1
kind: Pod
metadata:
  name: tester
  labels:
    robot: tester
spec:
  serviceAccount: jenkins-agent
  containers:
  - name: jnlp
  - name: ubuntu
    image: ubuntu:22.04
    tty: true
    command:
    - cat
"""
                }
            }
            steps {
                echo 'Testing the deployemnt with curl'
                // TODO: За допомогою контейнера ubuntu встановіть `curl`
                // TODO: Використайте curl, щоб зробити запит на http://labfive:80
                // TODO: Можливо, вам доведеться почекати приблизно 10 секунд, поки все буде розгорнуто вперше
            }
        }
    }
}
