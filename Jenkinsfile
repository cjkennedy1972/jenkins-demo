podTemplate(yaml: """
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: kustomize
    image: harbor.sixwords.dev/library/jenkins/kustomize:v3.4.0
    command:
    - cat
    tty: true
    env:
    - name: IMAGE_TAG
      value: ${BUILD_NUMBER}
    volumeMounts:
    - name: harbor-config
      mountPath: /root/.docker
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
    env:
    - name: IMAGE_TAG
      value: ${BUILD_NUMBER}
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug-539ddefcae3fd6b411a95982a830d987f4214251
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
    env:
    - name: DOCKER_CONFIG
      value: /root/.docker/
    - name: IMAGE_TAG
      value: ${BUILD_NUMBER}
    volumeMounts:
      - name: harbor-config
        mountPath: /root/.docker
  volumes:
    - name: harbor-config
      configMap:
        name: harbor-config
"""
  ) {

  node(POD_LABEL) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    stage('Build with Kaniko') {
      container('kaniko') {
        sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --skip-tls-verify --destination=harbor.sixwords.dev/library/jenkins/jenkins-demo:latest --destination=harbor.sixwords.dev/library/jenkins/jenkins-demo:v$BUILD_NUMBER'
      }
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Kustomize') {
      container('kustomize') {
        echo "3. Set Image"
        sh "kustomize edit set image harbor.sixwords.dev/jenkins/jenkins-demo:v$BUILD_NUMBER"
    }    
      stage('Deploy') {
        container('kubectl') { 
          echo "4. Deploy Pod"
          sh "kubectl apply -f k8s.yaml --record"
      }
   }
  }
 }
}
