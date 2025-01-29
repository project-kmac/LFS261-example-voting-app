pipeline {
	agent none

	stages{
		stage("worker-build"){
			when{
				changeset "**/worker/**"
			}
		
			agent {
				docker{
					image 'maven:3.9.8-sapmachine-21'
					args '-v $HOME/.m2:/root/.m2'
				}
			}

			steps{
				echo 'Compiling worker app'
				dir('worker'){
					sh 'mvn compile'
				}
			}

		}
		stage("worker-test"){
			when{
				changeset "**/worker/**"
			}

			agent {
				docker{
					image 'maven:3.9.8-sapmachine-21'
					args '-v $HOME/.m2:/root/.m2'
				}
			}
			steps{
				echo 'Running Unit Tests on worker app'
				dir('worker'){
					sh 'mvn clean test'
				}
			}
		}
		stage("worker-package"){
			when {
				branch 'master'
				changeset "**/worker/**"
			}
			agent {
				docker{
					image 'maven:3.9.8-sapmachine-21'
					args '-v $HOME/.m2:/root/.m2'
				}
			}
			steps{
				echo 'Packaging worker app'
				dir('worker'){
					sh 'mvn package -DskipTests'
					archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
				}
			}
		}
		stage("worker-docker-package"){
			agent any
			when{
				changeset "**/worker/**"
				branch 'master'
			}
			steps{
				echo 'Packaging worker app with docker'
				script{
					docker.withRegistry('https://index.docker.io/v1','dockerlogin') { def workerImage= docker.build("projectkmac/worker:v${env.BUILD_ID}", "./worker")
				workerImage.push()
				workerImage.push("latest")
}
				}
			}
		}

        stage('vote-build'){ 
            agent{
                docker{
                    image 'python:3.11-slim'
                    args '--user root'
                    }
                    }

            steps{ 
                echo 'Compiling vote app.' 
                dir('vote'){
            
                        sh "pip install -r requirements.txt"

                } 
            } 
        } 
        stage('vote-test'){ 
           
            agent {
                docker{
                    image 'python:3.11-slim'
                    args '--user root'
                    }
                    }
            steps{ 
                echo 'Running Unit Tests on vote app.' 
                dir('vote'){ 
                   
                        sh "pip install -r requirements.txt"
                        sh 'nosetests -v'
                        
                        
                } 
            } 
        } 

      stage('vote-docker-package'){
          agent any
         
          steps{
            echo 'Packaging wvoteorker app with docker'
            script{
              docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                  // ./vote is the path to the Dockerfile that Jenkins will find from the Github repo
                  def voteImage = docker.build("okapetanios/vote:v${env.BUILD_ID}", "./vote")
                  voteImage.push()
                  voteImage.push("${env.BRANCH_NAME}")
                  voteImage.push("latest")
              }
            }
          }
      }

		stage("result-build"){
			when{
				changeset "**/result/**"
			}
			steps{
				echo 'Compiling result app'
				dir('result'){
					sh 'npm install'
				}
			}

		}
		stage("result-test"){
			when{
				changeset "**/result/**"
			}
			steps{
				echo 'Running Unit Tests on result app'
				dir('result'){
					sh 'npm install && npm test'
				}
			}
		}
		stage("deploy to dev"{
			agent any
			when{
				branch 'master'
			}
			steps{
				echo 'Deploy instavote app with docker compose'
				sh 'docker compose up -d'
			}
		}

	}

}

post{
	always{
		echo 'Building multibranch pipeline for worker is completed..'
	}
}
