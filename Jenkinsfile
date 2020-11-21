node('martin-jnlp') {
    stage('Prepare') {
        echo "1.Prepare Stage"
        checkout scm
        echo "env.BRANCH_NAME ===>${env.BRANCH_NAME}"
        script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
      CI_REGISTRY = "registry-vpc.cn-beijing.aliyuncs.com"
      CI_REGISTRY_NAMESPACE = "nulizhu"
      CI_REGISTRY_IMAGE = "jenkins-demo"
      echo "3.Build Docker Image Stage"
      sh "docker build -t ${CI_REGISTRY}/${CI_REGISTRY_NAMESPACE}/${CI_REGISTRY_IMAGE}:${build_tag} ."
    }
    stage('Push') {
      echo "4.Push Docker Image Stage"
      withCredentials([usernamePassword(credentialsId: 'ali_docker_img_hub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
        sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword} ${CI_REGISTRY}"
        sh "docker push ${CI_REGISTRY}/${CI_REGISTRY_NAMESPACE}/${CI_REGISTRY_IMAGE}:${build_tag}"
      }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        sh "sed -i 's/<image>/${CI_REGISTRY}\\/${CI_REGISTRY_NAMESPACE}\\/${CI_REGISTRY_IMAGE}:${build_tag}/' k8s.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
        sh "kubectl apply -f k8s.yaml --record"
    }
}
