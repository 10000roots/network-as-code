// Jenkinsfile

pipeline {
    agent any // Run this on any available Jenkins agent

    stages {
        stage('Checkout Code') {
            steps {
                // This step automatically clones your GitHub repo
                checkout scm
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                // This is where Jenkins executes your playbook
                sh 'ansible-playbook -i inventory/hosts.yml playbooks/deploy_base_configs.yml'
            }
        }
    }
}