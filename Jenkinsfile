#!/usr/bin/groovy
def utils = new io.fabric8.Utils()

node {
  def envStage = utils.environmentNamespace('staging')
  def envProd = utils.environmentNamespace('production')

  git 'https://github.com/wangft/cakephp-ex.git'

  stage 'Canary release'
  echo 'NOTE: running pipelines for the first time will take longer as build and base docker images are pulled onto the node'
  if (!fileExists ('Dockerfile')) {
    writeFile file: 'Dockerfile', text: 'FROM fabric8/php:onbuild'
  }

  def newVersion = performCanaryRelease {}

    def rc = getKubernetesJson {
      port = 80
      label = 'php'
      icon = 'https://cdn.rawgit.com/fabric8io/fabric8/aebfa59/website/src/images/logos/php.png'
      version = newVersion
      imageName = clusterImageName
    }

  stage 'Rolling upgrade Staging'
  kubernetesApply(file: rc, environment: envStage)

  stage 'Approve'
  approve{
    room = null
    version = canaryVersion
    console = fabric8Console
    environment = envStage
  }

  stage 'Rolling upgrade Production'
  kubernetesApply(file: rc, environment: envProd)

}
