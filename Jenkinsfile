// Variables
def cloud = env.CLOUD ?: "kubernetes"
def registryCredsID = env.REGISTRY_CREDENTIALS ?: "nmicp"
def serviceAccount = env.SERVICE_ACCOUNT ?: "default"
def releaseName = env.RELEASE_NAME ?: "releaseName"
def namespace = env.NAMESPACE ?: "namespace"
def registry = env.REGISTRY ?: "nm-mgmt.iic.pl.ibm.com:8500"
def nodeSelector = env.NODE_SELECTOR ?: "beta.kubernetes.io/arch=amd64"

podTemplate(label: 'buildpod', cloud: cloud, serviceAccount: serviceAccount, namespace: namespace, nodeSelector: nodeSelector, envVars: [
        envVar(key: 'NAMESPACE', value: namespace),
        envVar(key: 'REGISTRY', value: registry),
        envVar(key: 'RELEASE_NAME', value: releaseName),
        envVar(key: 'NODE_SELECTOR', value: nodeSelector)
    ],
    volumes: [
        hostPathVolume(hostPath: '/etc/docker/certs.d', mountPath: '/etc/docker/certs.d'),
        hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
    ],
    containers: [
        containerTemplate(name: 'kubectl', image: 'ibmcom/k8s-kubectl:v1.8.3', ttyEnabled: true, command: 'cat'),
        containerTemplate(name: 'docker' , image: 'docker:17.06.1-ce', ttyEnabled: true, command: 'cat'),
    ]
) 
  
{
    node('buildpod') {
        checkout scm 
        container('docker') {
            stage('Build Docker Image in the Cloud') {
                sh """
                #!/bin/bash
                docker build -t ${env.REGISTRY}/${env.NAMESPACE}/michapp:${env.BUILD_NUMBER} .
                """
            }
            stage('Push Docker Image to Cloud Registry') {
                withCredentials([usernamePassword(credentialsId: registryCredsID,
                                               usernameVariable: 'USERNAME',
                                               passwordVariable: 'PASSWORD')]) {
                    sh """
                    #!/bin/bash
                    docker login -u ${USERNAME} -p ${PASSWORD} ${env.REGISTRY}
                    docker push ${env.REGISTRY}/${env.NAMESPACE}/michapp:${env.BUILD_NUMBER}
                    """
                }
            }
        }
        container('kubectl') {
            stage('Deploy New Application on the Cloud') {
                sh """
                SERVICE=`kubectl --namespace=${env.NAMESPACE} get service -l app=michapp,release=${env.RELEASE_NAME} -o name`
                kubectl --namespace=${env.NAMESPACE} get services
                if [ -z \${SERVICE} ]; then
                    # No service
                    echo 'Must create a service'
                    echo "Creating the service"
                    sed -i 's/RELEASE_NAME/${env.RELEASE_NAME}/g' michapp-svc.yaml
                    kubectl apply -f michapp-svc.yaml
                fi
                echo 'Service created'                
                kubectl --namespace=${env.NAMESPACE} describe service -l app=michapp,release=${env.RELEASE_NAME}

                DEPLOYMENT=`kubectl --namespace=${env.NAMESPACE} get deployments -l app=michapp,release=${env.RELEASE_NAME} -o name`
                kubectl --namespace=${env.NAMESPACE} get deployments             
                if [ -z \${DEPLOYMENT} ]; then
                    # No deployment to update
                    echo 'No deployment to update'
                    echo "Starting deployment"
                    sed -i 's/BUILD_NUMBER/${env.BUILD_NUMBER}/g' michapp-deploy.yaml
                    sed -i 's/RELEASE_NAME/${env.RELEASE_NAME}/g' michapp-deploy.yaml
                    kubectl apply -f michapp-deploy.yaml
                    exit 0
                fi
                kubectl --namespace=${env.NAMESPACE} set image \${DEPLOYMENT} michapp=${env.REGISTRY}/${env.NAMESPACE}/michapp:${env.BUILD_NUMBER}
                kubectl --namespace=${env.NAMESPACE} rollout status \${DEPLOYMENT}
                """
            }
        }
    }
}
