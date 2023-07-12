1. Set-up
    
    ```bash
    // 1. start the cluster
    ./hack/local-up-cluster.sh
    
    // 2. create a deployment
    ./cluster/kubectl.sh create -f test/jdtests/policies/InPlacePreferred/deployment.yaml
    
    // 3. describe the pod
    ./cluster/kubectl.sh get pods
    ./cluster/kubectl.sh describe pods e2e-test-***
    
    ```
    

1. Issue1: When describe the pods, found:
    
    ```bash
    Error response from daemon: Get "https://k8s.gcr.io/v2/": net/http: request canceled while waiting for connection
    ```
    
    can not connect to the google registery. use another one, for example:
