def app_name = "***"
def image_name = "uhub.service.ucloud.cn/abc/${tag}${app_name}:${BUILD_NUMBER}"
def docker_registry_auth = "e6250159-8d38-4ea8-8338-eb87d0fa8a83"
def registry = "uhub.service.ucloud.cn"
podTemplate(containers: [
    containerTemplate(name: 'kubectl', image: 'uhub.service.ucloud.cn/abc/kubectl:latest', command: 'cat', ttyEnabled: true),
  ],
  volumes: [
    hostPathVolume(mountPath: '/usr/bin/istioctl', hostPath: '/usr/bin/istioctl'),
  ],
  yaml: """\
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: uhub.service.ucloud.cn/abc/executor:debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
    - name: kaniko-secret
      secret:
        secretName: registrycred
    """.stripIndent()
  ) {
    node(POD_LABEL) {
        stage('Clone') {
            echo "拉取代码"
            checkout([$class: 'SubversionSCM',
                additionalCredentials: [],
                excludedCommitMessages: '',
                excludedRegions: '', excludedRevprop: '',
                excludedUsers: '',
                filterChangelog: false,
                ignoreDirPropChanges: false,
                includedRegions: '',
                locations: [[cancelProcessOnExternalsFail: true, credentialsId: '21b30458-ef58-4a2e-8962-05d68bb01adc', depthOption: 'infinity', ignoreExternalsOption: true, local: '.', remote: 'http://codeurl']], quietOperation: true, workspaceUpdater: [$class: 'UpdateUpdater']])
        }
        stage('Build Image') {
            container('kaniko') {
				script {
                  if (tag == 'test') {
				    sh "tar zvcf webroot.tar.gz *"
                    sh "mv   test/run.sh  test/jiacrontabd.ini test/default.conf test/api.exp  test/main-local.php  test/params-local.php . && sed -i '/admin_addr/s/jiacrontabadmin/testjiacrontabadmin/' jiacrontabd.ini && /kaniko/executor -c `pwd`/ -f `pwd`/test/Dockerfile-kaniko -d ${image_name}"
                  }
                  if (tag == 'pre') {
				    sh "tar zvcf webroot.tar.gz *"
                    sh "mv test/run.sh  test/jiacrontabd.ini test/default.conf test/api.exp . && sed -i '/admin_addr/s/jiacrontabadmin/prejiacrontabadmin/' jiacrontabd.ini && sed -i '/boardcast_addr/s/spider/prespider/' jiacrontabd.ini && /kaniko/executor -c `pwd`/ -f `pwd`/test/Dockerfile-kaniko -d ${image_name}"
                  }
                }    
            }
        }
        stage('Deploy') {
         withCredentials([file(credentialsId: 'e1a5edb3-926a-43d7-a79f-21c27484f7ed', variable: 'KUBECONFIG')]){
           container('kubectl') {
           //sh """
           //   mkdir -p ~/.kube && cp ${KUBECONFIG} ~/.kube/config
           //   sed -i "s#<IMAGE_NAME>#${image_name}#" ./test/deploy.yaml
			//  kubectl apply -f ./test/deploy.yaml
           // """
			script {
		  if (tag == 'test') {
				sh """
				sed -i "s#<IMAGE_NAME>#${image_name}#" ./test/php-deploy-utest.yaml
				#kubectl apply -f ./test/php-deploy-upre.yaml
				/usr/bin/istioctl kube-inject -f test/php-deploy-utest.yaml |kubectl apply -f -
				"""
			}
		  if (tag == 'pre') {
				sh """
				sed -i "s#<IMAGE_NAME>#${image_name}#" ./test/php-deploy-upre.yaml
				#kubectl apply -f ./test/php-deploy-upre.yaml
				/usr/bin/istioctl kube-inject -f test/php-deploy-upre.yaml |kubectl apply -f -
				"""
			}
		  
		}
            
           }
         }
        }
    }
}
