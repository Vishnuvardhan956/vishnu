def buildNumber = Jenkins.instance.getItem('egc-testing').lastSuccessfulBuild.number

pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: ubuntu
            image: harbor.codilar.dev/codilar/ubuntu:git
            command:
            - cat
            tty: true
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: kubernetes.io/hostname
                        operator: In
                        values:
                          - server4            
        '''
    }
  }
    environment {
    GIT_CREDS = credentials('codilar_gitlab')
    tag_id = "magento_tag: ${buildNumber}"
  }
    stages {
    stage('Clone') {
      steps {
        container('ubuntu') {
          git branch: 'master', credentialsId: 'codilar_gitlab', url: 'https://gitlab.codilar.in/devops/egc-testing.git' 
          echo "last egcsupply build_number: ${buildNumber}"
            }
        }
        }
        stage('image version change') {
      steps {
        container('ubuntu') {
          sh '''
                  #!/bin/bash
                sed -i  "s|magento_tag.*|${tag_id}|g" values.yaml
                git config --global --add safe.directory /home/jenkins/agent/workspace/egc-testing-cd/
                git config --global user.email "gopi.k@codilar.com"
                git config --global user.name "gopi"
                git add .
                git commit -m "tag change"
                git push https://${GIT_CREDS_USR}:${GIT_CREDS_PSW}@gitlab.codilar.in/devops/egc-testing.git master
             '''
        }
      }
     }        
    }

}
