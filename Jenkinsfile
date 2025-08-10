@Library('Shared') _

pipeline {
    agent any
    
    options {
        disableConcurrentBuilds()       //prevents two builds of this same job from running at once
        timestamps()                     //adds timestamps to log lines
    }

    environment {
        // Update the main app image name to match the deployment file
        DOCKER_IMAGE_NAME = 'trainwithshubham/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'trainwithshubham/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        GITHUB_CREDENTIALS = credentials('github-credentials')
        GIT_BRANCH = "master"
    }
    
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    clean_ws()
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                script {
                    clone("https://github.com/LondheShubham153/tws-e-commerce-app.git","master")
                }
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'Dockerfile',
                                context: '.'
                            )
                        }
                    }
                }
                
                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.'
                            )
                        }
                    }
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests()
                }
            }
        }
        
        stage('Security Scan with Trivy') {
            steps {
                script {
                    // Create directory for results
                  
                    trivy_scan()
                    
                }
            }
        }
        
        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }
                
                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: 'docker-hub-credentials'
                            )
                        }
                    }
                }
            }
        }
        
        // Add this new stage
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        gitCredentials: 'github-credentials',
                        gitUserName: 'Jenkins CI',
                        gitUserEmail: 'shubhamnath5@gmail.com'
                    )
                }
            }
        }
    }
}

pipeline {
  agent any
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

// PIPELINE: INCLUDING TERRAFORM IN CICD

  parameters {
    choice(name: 'TARGET_ENV', choices: ['dev', 'stage', 'prod'], description: 'Which environment to act on')   //Selects which environment’s Terraform code to operate on
    booleanParam(name: 'APPLY', defaultValue: false, description: 'Apply the planned changes?')                //Controls whether this run should actually APPLY or just PLAN
    string(name: 'TF_DIR', defaultValue: 'envs/dev/app', description: 'Path to Terraform working dir for this env') //This gets auto-adjusted later based on TARGET_ENV
  }

  environment {
    TF_IN_AUTOMATION = 'true'
    TF_INPUT         = 'false'
    APP_NAME = 'my-app'
    STAGING_NAMESPACE = 'staging'
    PROD_NAMESPACE = 'production'
    DOCKER_IMAGE_NAME = "my-registry/${APP_NAME}"
    DOCKER_IMAGE_TAG = "v${env.BUILD_NUMBER}"
  }

  stages {
    
    stage('Set Cloud Credentials') {
      steps {
        script {
          // Preferred: Jenkins agent with instance profile or IRSA (no static keys).
          // If you must use stored creds: add “AWS credentials” in Jenkins and:
          // withAWS(role: 'arn:aws:iam::123456789012:role/jenkins-tf-${TARGET_ENV}', roleSessionName: "jenkins-${env.BUILD_NUMBER}", duration: 3600) { ... }
        }
      }
    }

    stage('Init & Validate') {
      steps {
        dir("${env.TF_DIR}") {
          sh 'terraform fmt -check -recursive'            //Tfm format
          sh 'terraform init -lock-timeout=5m'            //Tfm init
          sh 'terraform validate'                        // Tfm validate
        }
      }
    }

    stage('Plan') {
      steps {
        dir("${env.TF_DIR}") {
          // serialize plans/applies per env to avoid overlapping locks
          lock(resource: "tf-${params.TARGET_ENV}") {
            sh 'terraform plan -lock-timeout=5m -out=plan.bin'                //stores Tfm PLAN output in a Binary file
            sh 'terraform show -no-color plan.bin > plan.txt'                //produces a human-readable file for review
          }
        }
          //Both plan.bin and plan.txt are archived as build artifacts so reviewers don’t have to download artifacts at later point of time
        archiveArtifacts artifacts: "${env.TF_DIR}/plan.bin, ${env.TF_DIR}/plan.txt", onlyIfSuccessful: true    
      }
        
      post {
        success {
          script {
            def plan = readFile("${env.TF_DIR}/plan.txt")                            //reading and displaying the PLAN file
            echo "---- Terraform Plan (truncated) ----\n" + plan.take(2000)
          }
        }
      }
    }

    stage('Approval (Prod only)') {
      when {
        allOf {                                            //block executes only when ALL OF below conditions are true
          expression { params.APPLY == true }
          expression { params.TARGET_ENV == 'prod' }
        }
      }
      steps {
        timeout(time: 2, unit: 'HOURS') {                    //min 2hrs required for this task, fails the build if no one approves in time
          input(
            message: "Approve Terraform APPLY to PROD?",            // need manual approval in Blue Ocean
            ok: "Approve & Continue",
            submitter: "tf-approvers,ops-leads"                 //restricts who can approve this, mapped to to Jenkins users/roles/groups
          )
        }
      }
    }

    stage('Apply') {
      when { expression { params.APPLY == true } }
      steps {
          dir("${env.TF_DIR}") {                                                // to ensure we’re applying the exact PLAN from this build
          lock(resource: "tf-${params.TARGET_ENV}") {
            copyArtifacts(projectName: env.JOB_NAME, selector: specific("${env.BUILD_NUMBER}"), filter: "${env.TF_DIR}/plan.bin")        // pulls the exact plan.bin created earlier by this same build number build
            sh 'ls -al'
            sh 'terraform apply -lock-timeout=5m -auto-approve plan.bin'        //Tfm APPLY command executes on Plan.bin which was reviewed
            sh 'terraform output -json > outputs.json || true'                    //Tfm Outputs are exported to outputs.json
          }
        }
      }
    }
  }

  post {
    success {
      archiveArtifacts artifacts: "${env.TF_DIR}/outputs.json", allowEmptyArchive: true        // archives outputs for traceability/integration
      echo "Done. Plan and any outputs archived."
    }
    always {
      // Optionally notify Slack/Teams, attach plan summary, etc.        
    }
  }
}


----------------------------------------------------------------------------------------------------------------------------------------------------------
//PIPELINE: INCLUDING KUBERNETES DEPLOYMENT HEALTHCHECK IN CICD -> can be done in 3 Steps
    
    stage('Health Check') {
            steps {
                script {
                    echo "Starting health checks for deployment ${APP_NAME} in namespace ${DEV_NAMESPACE}..."

                    // 1. Wait for the rollout to complete successfully. This is the most reliable way to check deployment health.
                    sh "kubectl rollout status deployment/${APP_NAME} -n ${STAGING_NAMESPACE} --timeout=5m"    /// this step will exit with an error if the rollout fails
                    echo "Deployment rollout completed successfully."

                    // 2. Check that pods are ready and running
                    // We use `kubectl get pods` and a simple grep/wc to count the number of ready pods
                    def desired_replicas = 3                                                     // Assuming you have 3 replicas
                    sh """
                        # Loop until the desired number of pods are ready
                        count=0
                        while [ \$count -le ${desired_replicas} ]; do
                            count=\$(kubectl get pods -l app=${APP_NAME} -n ${DEV_NAMESPACE} -o jsonpath='{range .items[*]}{.status.containerStatuses[*].ready}{" "}{end}' | grep -o true | wc -l)
                            echo "\$count out of ${desired_replicas} pods are ready..."
                            if [ \$count -eq ${desired_replicas} ]; then
                                echo "All ${desired_replicas} pods are ready."
                                break
                            fi
                            sleep 10
                        done
                    """

                    // 3. Verify the service is available and has endpoints
                    sh "kubectl describe service ${APP_NAME} -n ${DEV_NAMESPACE} | grep 'Endpoints:'"
                    echo "Service is available and has active endpoints."
                }
            }
        }
