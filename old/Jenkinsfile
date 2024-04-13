// Variables
def cloud = env.CLOUD ?: "kubernetes"
def registryCredsID = env.REGISTRY_CREDENTIALS ?: "clusterreg"
def serviceAccount = env.SERVICE_ACCOUNT ?: "jenk-jenkins"
def appname = env.APPNAME ?: "appname"
def namespace = env.NAMESPACE ?: "namespace"
def registry = env.REGISTRY ?: "nm-mgmt.iic.pl.ibm.com:8500"
def nodeSelector = env.NODE_SELECTOR ?: "beta.kubernetes.io/arch=ppc64le"

podTemplate(cloud: cloud, serviceAccount: serviceAccount, namespace: namespace, nodeSelector: nodeSelector, envVars: [
        envVar(key: 'NAMESPACE', value: namespace),
        envVar(key: 'REGISTRY', value: registry),
        envVar(key: 'NODE_SELECTOR', value: nodeSelector)
    ],
    volumes: [
        hostPathVolume(hostPath: '/etc/docker/certs.d', mountPath: '/etc/docker/certs.d'),
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
    ],
    containers: [
        containerTemplate(name: 'jnlp'   , image: 'nm-mgmt.iic.pl.ibm.com:8500/labns/jenkppc64-slave-jnlp:latest', args: '${computer.jnlpmac} ${computer.name}'),
        containerTemplate(name: 'kubectl', image: 'nm-mgmt.iic.pl.ibm.com:8500/labns/kubectl:v1.13.9', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker' , image: 'nm-mgmt.iic.pl.ibm.com:8500/labns/docker:latest', ttyEnabled: true, command: 'cat'),
    ]
) 
  
{
    node(POD_LABEL) {
        checkout scm 
        container('docker') {
            stage('Build Docker Image in the Cloud') {
                sh """
                #!/bin/bash
                sed -i 's/BUILD_NUMBER/${env.BUILD_NUMBER}/g' ${env.WORKSPACE}/index.html
                sed -i 's/APPNAME/${env.APPNAME}/g' ${env.WORKSPACE}/index.html
                docker build -t ${env.REGISTRY}/${env.NAMESPACE}/michapp-${env.APPNAME}:${env.BUILD_NUMBER} .
                """
            }
            stage('Push Docker Image to Cloud Registry') {
                withCredentials([usernamePassword(credentialsId: registryCredsID,
                                               usernameVariable: 'USERNAME',
                                               passwordVariable: 'PASSWORD')]) {
                    sh """
                    #!/bin/bash
                    docker login -u ${USERNAME} -p ${PASSWORD} ${env.REGISTRY}
                    docker push ${env.REGISTRY}/${env.NAMESPACE}/michapp-${env.APPNAME}:${env.BUILD_NUMBER}
                    """
                }
            }
        }
        container('kubectl') {
            stage('Deploy New Application on the Cloud') {
                sh """
                SERVICE=`kubectl --namespace=${env.NAMESPACE} get service -l app=michapp-${env.APPNAME} -o name`
                kubectl --namespace=${env.NAMESPACE} get services
                if [ -z \${SERVICE} ]; then
                    # No service
                    echo 'Must create a service'
                    echo "Creating the service"
                    sed -i 's/BUILD_NUMBER/${env.BUILD_NUMBER}/g' michapp-svc.yaml
                    sed -i 's/APPNAME/${env.APPNAME}/g' michapp-svc.yaml
                    sed -i 's/NAMESPACE/${env.NAMESPACE}/g' michapp-svc.yaml
                    kubectl apply -f michapp-svc.yaml
                fi
                echo 'Service created'                
                kubectl --namespace=${env.NAMESPACE} describe service -l app=michapp-${env.APPNAME}

                INGRESS=`kubectl --namespace=${env.NAMESPACE} get ingress -l app=michapp-${env.APPNAME} -o name`
                kubectl --namespace=${env.NAMESPACE} get ingress             
                if [ -z \${INGRESS} ]; then
                    # No ingress
                    echo 'Must create an ingress'
                    echo "Creating the ingress"
                    sed -i 's/BUILD_NUMBER/${env.BUILD_NUMBER}/g' michapp-ing.yaml
                    sed -i 's/APPNAME/${env.APPNAME}/g' michapp-ing.yaml
                    kubectl apply -f michapp-ing.yaml
                fi

                DEPLOYMENT=`kubectl --namespace=${env.NAMESPACE} get deployments -l app=michapp-${env.APPNAME} -o name`
                kubectl --namespace=${env.NAMESPACE} get deployments        
                if [ -z \${DEPLOYMENT} ]; then
                    # No deployment to update
                    echo 'No deployment to update'
                    echo "Starting deployment"
                    sed -i 's/BUILD_NUMBER/${env.BUILD_NUMBER}/g' michapp-deploy.yaml
                    sed -i 's/APPNAME/${env.APPNAME}/g' michapp-deploy.yaml
                    sed -i 's/NAMESPACE/${env.NAMESPACE}/g' michapp-deploy.yaml
                    kubectl apply -f michapp-deploy.yaml
                    exit 0
                fi
                kubectl --namespace=${env.NAMESPACE} set image \${DEPLOYMENT} michapp-${env.APPNAME}=${env.REGISTRY}/${env.NAMESPACE}/michapp-${env.APPNAME}:${env.BUILD_NUMBER}
                kubectl --namespace=${env.NAMESPACE} rollout status \${DEPLOYMENT}
                """
            }
        }
    }
}
