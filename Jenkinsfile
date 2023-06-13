pipeline {
    agent { label 'iti' }
    parameters {
        choice(name: 'ENV', choices: ['dev', 'test', 'prod',"release"])
    } 
    stages {
        stage('build') {
            steps {
                echo 'build'
                script{
                    if (params.ENV == "release") {
                        withCredentials([usernamePassword(credentialsId: 'iti-dockerhub', usernameVariable: 'USERNAME_ITI', passwordVariable: 'PASSWORD_ITI')]) {
                            sh '''
                                 docker login -u ${USERNAME_SYSADMIN} -p ${PASSWORD_SYSADMIN}
                                docker build -t 12345676700/bakehouseitisysadmin:v${BUILD_NUMBER} .
                                docker push 12345676700/bakehouseitisysadmin:v${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../build_num.txt
                                echo ${ENV_ITI}
                                
                            '''
                        }
                    }
                    else {
                        echo "user choosed ${params.ENV}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                echo 'deploy'
                script {
                    if (params.ENV == "dev" || params.ENV == "test" || params.ENV == "prod") {
                        withCredentials([file(credentialsId: 'iti-kubeconfig', variable: 'KUBECONFIG_ITI')]) {
                            sh '''
                                export BUILD_NUMBER=$(cat ../build.txt)
                                mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                                cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                                rm -f Deployment/deploy.yaml.tmp
                                kubectl apply -f Deployment --kubeconfig ${KUBECONFIG_ITI} -n ${ENV}
                            '''
                        }
                    }
                }
            }
        }
    }
}
