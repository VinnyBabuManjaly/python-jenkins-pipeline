#!groovy

pipeline {
    agent {
        docker {
            image 'python:3.7'
        }
    }

    stages {
        stage('Environment preparation') {
            steps {
                echo "-=- preparing project environment -=-"
                // Python dependencies
                bat "pip install -r requirements.txt"
            }
        }
        stage('Compile') {
            steps {
                echo "-=- compiling project -=-"
                bat "python -m compileall ."
            }
        }

        stage('Unit tests') {
            steps {
                echo "-=- execute unit tests -=-"
                bat "nosetests -v test"
            }
        }

        stage('Mutation tests') {
            steps {
                echo "-=- execute mutation tests -=-"
                // initialize mutation testing session
                bat "cosmic-ray init config.yml jenkins_session && cosmic-ray --verbose exec jenkins_session && cosmic-ray dump jenkins_session | cr-report"    
            }
        }

        stage('Package') {
            steps {
                echo "-=- packaging project -=-"
                echo "No packaging phase for python projects ..."
            }
        }

        stage('Build Docker image') {
            steps {
                echo "-=- build Docker image -=-"
                bat "docker build -t restalion/python-jenkins-pipeline:0.1 ."
            }
        }

        stage('Run Docker image') {
            steps {
                echo "-=- run Docker image -=-"
                bat "docker run --name python-jenkins-pipeline --detach --rm --network ci -p 5001:5000 restalion/python-jenkins-pipeline:0.1"
            }
        }

        stage('Integration tests') {
            steps {
                echo "-=- execute integration tests -=-"
                bat "nosetests -v int_test"
            }
        }

        stage('Performance tests') {
            steps {
                echo "-=- execute performance tests -=-"
                bat "locust -f ./perf_test/locustfile.py --no-web -c 1000 -r 100 --run-time 1m -H http://172.18.0.3:5001"
            }
        }

        stage('Dependency vulnerability tests') {
            steps {
                echo "-=- run dependency vulnerability tests -=-"
                bat "safety check"
            }
        }

        stage('Code inspection & quality gate') {
            steps {
                echo "-=- run code inspection & quality gate -=-"
                bat "pylama"
            }
        }

        stage('Push Docker image') {
            steps {
                echo "-=- push Docker image -=-"
                withDockerRegistry([ credentialsId: "werdar-wedartg-uiny67-adsuja0-12njkn3", url: "" ]) {
                    bat "docker push restalion/python-jenkins-pipeline:0.1"
                }
                
                //sh "mvn docker:push"
            }
        }
    }

    post {
        always {
            echo "-=- remove deployment -=-"
            bat "docker stop python-jenkins-pipeline"
        }
    }
}
