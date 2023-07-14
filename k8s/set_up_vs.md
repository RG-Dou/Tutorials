
1. Set-up
    
    ```
    // 1. start the cluster
    ./hack/local-up-cluster.sh
    
    // 2. create a deployment
    ./cluster/kubectl.sh create -f test/jdtests/policies/InPlacePreferred/deployment.yaml
    
    // 3. describe the pod
    ./cluster/kubectl.sh get pods
    ./cluster/kubectl.sh describe pods e2e-test-***
    
    // 4. other scripts about vertical scaling, check the test/jdtests/policies/InPlacePreferred/tests.md
    
    ```
    
2. Issue1: not set go path:
    
    ```bash
    // version is
    wget https://dl.google.com/go/go1.10.2.linux-amd64.tar.gz
    tar -zvxf go1.10.2.linux-amd64.tar.gz
    mv go /usr/local/
    
    // set the local variable
    vim ~/.profile
    export PATH=$PATH:/usr/local/go/bin/
    // stop the fish
    source ~/.profile
    
    // Verify the go
    go version
    ```
    
3. Issue2: etcd not in the path
    
    ```bash
    sudo apt-get install etcd-server
    ```
    
4. Issue3: docker is run as the root, your user is not in the docker group
    
    ```bash
    sudo usermod -a -G docker drg
    // logout and re login
    ```
    
5. Issue1: When describe the pods, found:
    
    ```
    Error response from daemon: Get "<https://k8s.gcr.io/v2/>": net/http: request canceled while waiting for connection
    
    ```
    
    can not connect to the google registery. use another one, for example:
    docker pull [registry.aliyuncs.com/google_containers/pause:3.1](<http://registry.aliyuncs.com/google_containers/pause:3.1>)
    docker tag [registry.aliyuncs.com/google_containers/pause:3.1](<http://registry.aliyuncs.com/google_containers/pause:3.1>) [k8s.gcr.io/pause:3.](http://k8s.gcr.io/pause:3.1)


run k8s hello-samza
https://hub.docker.com/r/anaerobic/hello-samza
deployment.yaml在test/jdtests/policies/hello-samza

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hello-samza
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hello-samza
      annotations:
        oomKillDisable: "false"
        scheduler.alpha.kubernetes.io/resize-resources-policy: "InPlacePreferred"
#        scheduler.alpha.kubernetes.io/resize-resources-policy: "Restart"
    spec:
      hostNetwork: true
      containers:
      - name: hs1
        image: anaerobic/hello-samza:latest
        imagePullPolicy: "IfNotPresent"
#        imagePullPolicy: "Never"
        resources:
          requests:
            cpu: "4"
            memory: "4Gi"
          limits:
            cpu: "4"
            memory: "4Gi"
        command:
          - sleep
          - "100000000"

进入container内部编译：
./cluster/kubectl.sh exec -it hello-samza-* -- bash
需要等container启动1分钟之后再编译等
