pipeline {
    agent any 
    stages {
        stage('Build Phase -> Creating bc') { 
            when {
                expression {
                    openshift.withCluster() {
                        openshift.withProject('dev') {
                                return !openshift.selector("bc", "train-schedule").exists();
                        }
                    }
                }
            }
            steps{
                script {
                     openshift.withCluster() {
                        openshift.withProject('dev') {
                            openshift.newBuild("openshift/nodejs~https://github.com/afanrasool/cicd-pipeline-train-schedule-git.git", "--name=train-schedule")
                        }
                    }
                }
            }
        }
        stage('Build Phase --> Start Build') { 
            steps{
                script {
                     openshift.withCluster() {
                        openshift.withProject('dev') {
                            openshift.selector("bc", "train-schedule").startBuild("--wait=true")
                        }
                    }
                }
            sh("echo Image built and ready to be deployed")
            }
        }
        stage('Deploy Image in Test Project') { 
            when {
              expression {
                openshift.withCluster() {
                    openshift.withProject('dev') {
                                return !openshift.selector('dc', 'train-schedule').exists()
                            }
                        }
                    }
                }
                steps {
                    script {
                        openshift.withCluster() {
                            openshift.withProject('dev') {
                                def app = openshift.newApp("train-schedule", "--name=train-schedule")
                                app.narrow("svc").expose();
                                def dc = openshift.selector("dc", "train-schedule")
                                while (dc.object().spec.replicas != dc.object().status.availableReplicas) {
                                    sleep 10
                                }
                                openshift.set("triggers", "dc/train-schedule", "--manual")
                            }
                        }
                    }
                }
        }
    }
}
