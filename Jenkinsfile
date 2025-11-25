pipeline {
    agent { label 'jenkins-aks' }

    environment {
        ARM_CLIENT_ID        = credentials('ARM_CLIENT_ID')
        ARM_CLIENT_SECRET    = credentials('ARM_CLIENT_SECRET')
        ARM_TENANT_ID        = credentials('ARM_TENANT_ID')
        ARM_SUBSCRIPTION_ID  = credentials('ARM_SUBSCRIPTION_ID')

        RESOURCE_GROUP = "jenkins-poc"
        AKS_CLUSTER    = "jenkins"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Azure Login') {
            steps {
                sh '''
                    echo "Logging into Azure using Service Principal…"
                    az login --service-principal \
                        -u "$ARM_CLIENT_ID" \
                        -p "$ARM_CLIENT_SECRET" \
                        --tenant "$ARM_TENANT_ID"

                    echo "Setting subscription…"
                    az account set --subscription "$ARM_SUBSCRIPTION_ID"
                '''
            }
        }

        stage('Get AKS Credentials') {
            steps {
                sh '''
                    echo "Getting kubeconfig for AKS…"
                    az aks get-credentials \
                        -n "$AKS_CLUSTER" \
                        -g "$RESOURCE_GROUP" \
                        --overwrite-existing

                    echo "Converting kubeconfig using kubelogin (Azure CLI mode)…"
                    kubelogin convert-kubeconfig -l azurecli
                '''
            }
        }

        stage('Deploy Nginx App') {
            steps {
                sh '''
                    echo "Checking cluster access…"
                    kubectl get nodes

                    echo "Applying nginx-test.yaml…"
                    kubectl apply -f nginx-test.yaml
                    kubectl apply -f nginx-service.yaml

                    echo "Checking pods…"
                    kubectl get pods -o wide
                '''
            }
        }
    }
}
