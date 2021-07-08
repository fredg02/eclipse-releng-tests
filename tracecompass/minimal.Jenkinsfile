
pipeline {
    agent {
        kubernetes {
            label 'my-agent-pod'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:alpine
    command:
    - cat
    tty: true
  - name: jnlp
    image: 'eclipsecbi/jenkins-jnlp-agent'
    volumeMounts:
    - mountPath: /home/jenkins/.ssh
      name: volume-known-hosts
  volumes:
  - name: volume-known-hosts
    configMap:
      name: known-hosts
 """
            defaultContainer 'my-agent-pod'
        }
    }
    stages {
        stage('Deploy Site') {
            steps {
                container('jnlp') {
                    sshagent (['projects-storage.eclipse.org-bot-ssh']) {
                        sh 'ssh genie.tracecompass@projects-storage.eclipse.org ls -al'
                    }
                }
            }
        }
    }
}
