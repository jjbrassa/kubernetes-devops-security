//@Library('slack') _

pipeline {
  agent any

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "jjbrassa/numeric-app:${GIT_COMMIT}"
    applicationURL = "http://ec2-52-9-90-238.us-west-1.compute.amazonaws.com"
    applicationURI = "/increment/99"
  }

  stages {

      // ---------------------------------
      // Build the artifact
      // ---------------------------------
      stage('Build Artifact - Maven') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archiveArtifacts 'target/*.jar' 
            }
      }

      // ---------------------------------
      // Run the unit test suite
      // ---------------------------------
      stage('Unit Tests - Junit and JaCoco') {
        steps {
          sh "mvn test"
        }
      }

      // ---------------------------------
      // Run mutation tests
      // ---------------------------------
      stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
      }
      
      // ---------------------------------
      // SAST Tests with SonarCube
      // ---------------------------------
      stage('SonarQube - SAST') {
        steps {
          sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://ec2-52-9-90-238.us-west-1.compute.amazonaws.com:9000 -Dsonar.token=sqp_40641e5e4fc22039759549b02ce5505c6558a61b"
        }
      }
      
      // ---------------------------------
      // Docker build!
      // ---------------------------------
       stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
            sh 'printenv'
            sh 'sudo docker build -t jjbrassa/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push jjbrassa/numeric-app:""$GIT_COMMIT""'
          }
        }
      }

      // ---------------------------------
      // k8s Build - Dev env
      // ---------------------------------
      stage('K8S Deployment - DEV') {
        steps {
          parallel(
            "Deployment": {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "bash k8s-deployment.sh"
              }
            },
            "Rollout Status": {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "bash k8s-deployment-rollout-status.sh"
              }
            }
          )
        }
      }

  }
  // ---------------------------------
  // Move all the report output!
  // ---------------------------------
  post {
    always {
      junit 'target/surefire-reports/*.xml'
      jacoco execPattern: 'target/jacoco.exec'
      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
      //dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      //publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report', useWrapperFileDirectly: true])
      //sendNotification currentBuild.result
    }
  }
}
      // stage('Vulnerability Scan - Docker') {
      //   steps {
      //     parallel(
      //       "Dependency Scan": {
      //         sh "mvn dependency-check:check"
      //       },
      //       "Trivy Scan": {
      //         sh "bash trivy-docker-image-scan.sh"
      //       },
      //       "OPA Conftest": {
      //         sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
      //       }
      //     )
      //   }
      // }

      // stage('Vulnerability Scan - Kubernetes') {
      //   steps {
      //     parallel(
      //       "OPA Scan": {
      //         sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
      //       },
      //       "Kubesec Scan": {
      //         sh "bash kubesec-scan.sh"
      //       }
      //       //"Trivy Scan": {
      //       //  sh "bash trivy-k8s-scan.sh"
      //       //}
      //     )
      //   }
      // }

      // stage('Integration Tests - DEV') {
      //   steps {
      //     script {
      //       try {
      //         withKubeConfig([credentialsId: 'kubeconfig']) {
      //           sh "bash integration-test.sh"
      //         }
      //       } catch (e) {
      //         withKubeConfig([credentialsId: 'kubeconfig']) {
      //           sh "kubectl -n default rollout undo deploy ${deploymentName}"
      //         }
      //         throw e
      //       }
      //     }
      //   }
      // }  

      // stage('OWASP ZAP - DAST') {
      //   steps {
      //     withKubeConfig([credentialsId: 'kubeconfig']) {
      //       sh 'bash zap.sh'
      //     }
      //   }
      // }

    //   stage('Prompte to PROD?') {
    //     steps {
    //       timeout(time: 2, unit: 'DAYS') {
    //         input 'Do you want to Approve the Deployment to Production Environment/Namespace?'
    //       }
    //     }
    //   }
    // }


    // success {

    // }

    // failure {

    // }