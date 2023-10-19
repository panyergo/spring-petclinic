pipeline {
    environment {
    registry = "yergodav/demo"
    registryCredential = 'dockerID'
    TLCred = 'TLauth'
    dockerImage = ''
    }
    agent any
    tools {
        jdk "jdk17"
        maven "maven"
    }
    
    stages {
        stage ('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/panyergo/spring-petclinic.git'
            }
        }
        
        stage ('Code Compile') {
            steps {
                //sh 'mvn clean compile'
                sh 'mvn clean install'
            }
        }
        
        stage ('Unit Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage ('Build Image') {
            steps {
                script {
                def dockerImage = docker.build registry + ":$BUILD_NUMBER"
                dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
                sh 'docker save -o demo-$BUILD_NUMBER.tar ' +  registry + ":$BUILD_NUMBER"
            }
        }

        stage('Scan image with twistcli') {
            steps {
		withCredentials([usernamePassword(credentialsId: 'TLauth', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
                sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://console-master-demo2.dyergovich.demo.twistlock.com/api/v1/util/twistcli'
                sh 'pwd ; chmod a+x ./twistcli'
                sh "./twistcli images scan --u $TL_USER --p $TL_PASS --address https://console-master-demo2.dyergovich.demo.twistlock.com/ --details \"$registry:$BUILD_NUMBER\""
                sh 'rm ./twistcli'
		}
            }
        }

        stage('Scan Compute Plugin') {
            steps {
                // Scan the image
                prismaCloudScanImage ca: '',
                cert: '',
                dockerAddress: 'unix:///var/run/docker.sock',
                //image: 'test/test-image*',
                image: 'yergodav/demo*',
                key: '',
                logLevel: 'info',
                podmanPath: '',
                // The project field below is only applicable if you are using Prisma Cloud Compute Edition and have set up projects (multiple consoles) on Prisma Cloud.
                project: '',
                resultsFile: 'prisma-cloud-scan-results.json',
                ignoreImageBuildTime:true
            }
        }

        stage('Push Results') {
            steps {
                // push the results
                   prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
            }
        }
           
        stage('Push Image') {
            steps{
                script {
                dockerImage = docker.build registry + ":$BUILD_NUMBER"
                docker.withRegistry( '', registryCredential ) {
                //dockerImage.push()
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push("latest")
                }
                }
            }
        }
    }
}
