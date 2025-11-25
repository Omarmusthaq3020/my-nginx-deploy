pipeline {
    agent { label 'jenkins-aks' }

    environment {
        AZURE_SUBSCRIPTION = "aks1-sc"
        RESOURCE_GROUP     = "poc"
        AKS_CLUSTER        = "aks1"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Login to Azure') {
            steps {
                sh '''
                    echo "Logging into Azure…"
                    az login --service-principal \
                        -u $AZURE_CLIENT_ID \
                        -p $AZURE_CLIENT_SECRET \
                        --tenant $AZURE_TENANT_ID
                    
                    az account set --subscription "$AZURE_SUBSCRIPTION"
                '''
            }
        }

        stage('Get AKS Credentials') {
            steps {
                sh '''
                    echo "Fetching AKS kubeconfig…"
                    az aks get-credentials -n "$AKS_CLUSTER" -g "$RESOURCE_GROUP" --overwrite-existing

                    echo "Converting kubeconfig using kubelogin..."
                    kubelogin convert-kubeconfig -l azurecli
                '''
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh '''
                    echo "Testing AKS access…"
                    kubectl get nodes

                    echo "Applying nginx manifest…"
                    kubectl apply -f nginx-test.yaml

                    echo "Checking deployed pods…"
                    kubectl get pods -o wide
                '''
            }
        }
    }
}
