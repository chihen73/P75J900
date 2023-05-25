# P75J900 Final Project

2023 Spring COMPUTER SYSTEMS ARCHITECTURE AND BIG DATA PLATFORMS
Final Project Kubernetes **Operator Pattern**

## GKE create cluster

**Cloud Shell**
設定專案

```gcloud config set project [PROJECT_ID]```
```gcloud config set project lfclass-356014```

建立叢集

```gcloud container clusters create operator-demo-cluster --num-nodes 3 --machine-type n1-standard-1 --zone asia-east1-c```

## ConfigWatcher

[Operator example README](https://github.com/k8spatterns/examples/blob/main/advanced/Operator/README.adoc)

Enable shell autocompletion ans alias

`echo 'source <(kubectl completion bash)' >>~/.bashrc`

First, install the CRD and create a role that allows the modification of custom resources created by this CRD:

`kubectl apply -f https://k8spatterns.io/Operator/config-watcher-crd.yml`

Verify that the CRD has been registered:

`kubectl get crd | grep configwatchers`

The operator script is stored in a ConfigMap:

`wget https://raw.githubusercontent.com/k8spatterns/examples/main/advanced/Operator/config-watcher-operator.sh`

`kubectl create configmap config-watcher-operator --from-file=./config-watcher-operator.sh`

To create deployment in the current namespace, run the following:

`kubectl apply -f https://k8spatterns.io/Operator/config-watcher-operator.yml`

Before deploying the web app, open another terminal and tail the log of our Operator to see the events received by the Operator as they come in:

`kubectl logs -f $(kubectl get pods -l role=operator -o name) config-watcher`

Now, create the web application:

`kubectl apply -f https://raw.githubusercontent.com/chihen73/P75J900/main/web-app.yml`

Access the application:

`kubectl get svc`

Using EXTERNAL-IP and PORT in your browser for accessing

`<EXTERNAL-IP>:<PORT>`

To trigger the Operator on a config change, install a custom resource of the ConfigWatcher kind:

`kubectl apply -f https://k8spatterns.io/Operator/config-watcher-sample.yml`

Check that the resource has been created:

`kubectl get configwatchers`

After establishing the connection between the ConfigMap and Pods, change the content of the ConfigMap and watch the Operator log:

`kubectl patch configmap webapp-config -p '{"data":{"message":"Greets from your smooth operator!"}}'`
