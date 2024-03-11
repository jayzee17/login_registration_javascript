void setBuildStatus(String message, String state) {
  step([
      $class: "GitHubCommitStatusSetter",
      reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/jayzee17/login_registration_javascript.git"],
      contextSource: [$class: "ManuallyEnteredCommitContextSource", context: "ci/jenkins/build-status"],
      errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
      statusResultSource: [ $class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]] ]
  ]);
} 

applicationIPAddress= "54.179.251.186"
sourceBranch = ghprbSourceBranch
targetBranch = ghprbTargetBranch

pipeline {
    agent any
    //tools { nodejs "NodeJS" }

    // test 

    stages {
        stage('Deploy Sample Instance') {
            steps {
                script {
                    Integer port = 8080
                    String directory = "/var/www/sample_app"
                    String secret_helper = "secret_helper"

                    echo "port is ${port}"
                    echo "directory is ${directory}"
                    echo "staging_env is ${secret_helper}"
                  

                    withCredentials([sshUserPrivateKey(credentialsId: "ssh-credentials", keyFileVariable: 'SSH_KEY')]) {
                        def remote = [
                            name: 'ubuntu',
                            port: 22,
                            allowAnyHosts: true,
                            host: "${applicationIPAddress}",
                            user: "ubuntu",
                            identityFile: SSH_KEY
                        ]

                        echo "Fetch branch and checkout to change branch"
                        sshCommand remote: remote, command: "cd ${directory} && sudo git fetch"
                        sshCommand remote: remote, command: "cd ${directory} && sudo git checkout ${sourceBranch}"
                        sshCommand remote: remote, command: "cd ${directory} && sudo git pull origin ${sourceBranch}"

                        withCredentials([file(credentialsId: secret_helper, variable: 'yaml_file')]) {
                            sh 'mv \$yaml_file ./configs'
                            sshPut remote: remote, from: "./configs/sample.env.yml", into: "/var/www/tmp_server_files/"
                        }

                        sshCommand remote: remote, command: "sudo rm -rf ${directory}/configs/sample.env.yml"
                        sshCommand remote: remote, command: "sudo mv /var/www/tmp_server_files/sample.env.yml ${directory}/configs/"
                    }

                    echo currentBuild.result
                }
            }
        }
    } 
}
