Kubernetes Custom Metrics Adapter for Prometheus
================================================

作用
-------------
作为自定义的metrics的api适配器，组合上prometheus，可以先提供出一系列的metrics。  
根据custom-metrics-apiserver，可以编写出自定义的metrics，结合进来，提供更丰富的各类metrics。

使用
-------------

1. 修改
    * Makefile    
        ```REGISTRY```修改成docker registry的名称
    * 创建镜像  
        ```make docker-build```创建对应镜像，然后手动push到对应的docker registry
    * 修改adapter deploy
        ```vim ./deploy/manifests/custom-metrics-apiserver-deployment.yaml```修改对应的docker image

2. 部署
    * 创建namespace  
        ```kubectl apply -f namespace.yaml```  
        * 如果改namespace，则需要修改对应的yaml
        * prometheus的ns->monitoring
        * 自定义的metrics的ns->custom-metrics
    * 创建mertics-server  
        ```kubectl create -f ./deploy/metrics-server/deploy/1.8+```
    * 部署prometheus  
        ```kubectl create -f ./deploy/prometheus```
    * 创建cert  
        ```cd deploy && make certs```
    * 部署adapter  
        tip: 部署之前需要创建cert,并且对应的manifests下的dep.yaml里的image已经push  
        ```kubectl create -f ./deploy/manifests```
    * 查看prometheus提供的一系列metrics   
        ```kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1" | jq .```
    * 查看ns为monitoring下的pods的FS_usage  
        ```kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/monitoring/pods/*/fs_usage_bytes" | jq .```

3. 自定义metrics测试
    * 创建一个[部署sample-app](./deploy/sample-app/sample-app.deploy.yaml)，并公开端口用于接收请求，提升metrics  
        ```kubectl create -f sample-app.deploy.yaml```
        ```kubectl create service clusterip sample-app --tcp=80:8080```
    * demo提供了一些关于接收http请求的数量metrics，http_requests_totle，查询
        ```kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests?selector=app%3Dsample-app"```
    * 增加请求
        ```curl http://$(kubectl get service sample-app -o jsonpath='{ .spec.clusterIP }')/metrics```
    * 创建[sample-app的HPA](./deploy/sample-app/sample-app-hpa.yaml)
        ```kubectl create -f sample-app-hpa.yaml```

案例参考
-------------

[docs/walkthrough.md](docs/walkthrough.md).




