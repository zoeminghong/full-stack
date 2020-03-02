# Kubernetes

```
# 启动容器程序
kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080

#列表已经部署的应用
kubectl get deployments
# 创建代理
echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n"; kubectl proxy

export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')

curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
```

