pipeline {
    agent {
       kubernetes {
           yaml """
apiVersion: v1 
kind: Pod 
metadata: 
    name: dind
    namespace: jenkins
    annotations:
      container.apparmor.security.beta.kubernetes.io/dind: unconfined
      container.seccomp.security.alpha.kubernetes.io/dind: unconfined
spec: 
    nodeSelector:
      topology.kubernetes.io/zone: eu-west-1c
    nodeName: ip-10-0-209-124.eu-west-1.compute.internal
    containers: 
      - name: dind
        image: docker:dind
        securityContext:
          privileged: true
        tty: true
        volumeMounts:
        - name: var-run
          mountPath: /var/run
        - name: jenkins-cache
          mountPath: /var/lib/docker
      - name: jnlp
        securityContext:
          runAsUser: 0
          fsGroup: 0
        volumeMounts:
        - name: var-run
          mountPath: /var/run
        - name: jenkins-cache
          mountPath: /var/lib/docker
        
    volumes:
    - name: var-run
      emptyDir: {}
    - name: jenkins-cache
      hostPath: 
        path: /tmp/cache
        type: DirectoryOrCreate
"""
       }
   }

    parameters { 
        string(name: 'DOCKER_REPOSITORY', defaultValue: 'sysdigcicd/cronagent', description: 'Name of the image to be built (e.g.: sysdiglabs/dummy-vuln-app)') 
    }
    
    environment {
        DOCKER = credentials('docker-repository-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                container("dind") {
                    checkout scm
                }
            }
        }
        stage('Build Image') {
            steps {
                container("dind") {
                    sh "docker build -f Dockerfile -t ${params.DOCKER_REPOSITORY} ."
                    sh "echo ${params.DOCKER_REPOSITORY} > sysdig_secure_images"
                }
            }
        }
        stage('Scanning Image') {
            steps {
                // This will always be executed in the JNLP container
                sysdig engineCredentialsId: 'sysdig-secure-api-credentials', name: 'sysdig_secure_images', inlineScanning: true
            }
        }
   }
}
