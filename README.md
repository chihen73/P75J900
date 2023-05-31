# P75J900 Final Project

2023 Spring COMPUTER SYSTEMS ARCHITECTURE AND BIG DATA PLATFORMS<br>
Final Project **Kubernetes Operator Pattern**<br>
[Project github Repo](https://github.com/chihen73/P75J900)

## GKE create cluster

### Cloud Shell

[Reset Google CloudShell](https://cloud.google.com/shell/docs/resetting-cloud-shell)

Setting project<br>
`gcloud config set project [PROJECT_ID]`<br>
`gcloud config set project lfclass-356014`

Create Cluster<br>
`gcloud container clusters create operator-demo-cluster --num-nodes 3 --machine-type n1-standard-1 --zone asia-east1-c --cluster-version 1.26.3-gke.1000`

cluster version<br>
`1.23.17-gke.1700`<br>
`1.24.11-gke.1000`<br>
`1.25.8-gke.500`<br>
`1.26.3-gke.1000`

Enable shell autocompletion ans alias<br>
`echo 'source <(kubectl completion bash)' >>~/.bashrc`

## ConfigWatcher

[Operator example README](https://github.com/k8spatterns/examples/blob/main/advanced/Operator/README.adoc)<br>

Watching resources

`watch -n 1 kubectl get cw,pods,configmaps -o wide`

### Create CRD

First, install the CRD and create a role that allows the modification of custom resources created by this CRD:

`kubectl apply -f https://k8spatterns.io/Operator/config-watcher-crd.yml`

Verify that the CRD has been registered:

`kubectl get crd | grep configwatchers`

### Create Operator

The operator script is stored in a ConfigMap:

`wget https://raw.githubusercontent.com/k8spatterns/examples/main/advanced/Operator/config-watcher-operator.sh`

`kubectl create configmap config-watcher-operator --from-file=./config-watcher-operator.sh`

To create deployment in the current namespace, run the following:

`kubectl apply -f https://k8spatterns.io/Operator/config-watcher-operator.yml`

Before deploying the web app, open another terminal and tail the log of our Operator to see the events received by the Operator as they come in:

`kubectl logs -f $(kubectl get pods -l role=operator -o name) config-watcher`

### Create WebApp

Now, create the web application:

`kubectl apply -f https://raw.githubusercontent.com/chihen73/P75J900/main/ConfigWatcher/web-app.yml`

Access the application:

`kubectl get svc`

Using EXTERNAL-IP and PORT in your browser for accessing

`<EXTERNAL-IP>:<PORT>`

### Create CRD Instance

To trigger the Operator on a config change, install a custom resource of the ConfigWatcher kind:

`kubectl apply -f https://k8spatterns.io/Operator/config-watcher-sample.yml`

Check that the resource has been created:

`kubectl get cw,pods,configmaps`

### Testing

After establishing the connection between the ConfigMap and Pods, change the content of the ConfigMap and watch the Operator log:

`kubectl patch configmap webapp-config -p '{"data":{"message":"Greets from your smooth operator!"}}'`

### Clean-up

```
kubectl delete -f https://k8spatterns.io/Operator/config-watcher-sample.yml
kubectl delete -f https://raw.githubusercontent.com/chihen73/P75J900/main/ConfigWatcher/web-app.yml
kubectl delete -f https://k8spatterns.io/Operator/config-watcher-operator.yml<br>
kubectl delete configmap config-watcher-operator
kubectl delete -f https://k8spatterns.io/Operator/config-watcher-crd.yml
```
---

## Develop an Operator with Kubebuilder

### The at Command
Using Cloud Shell access to GCP VM<br>

Install at command<br>
`sudo apt update`<br>
`sudo apt install at`<br>
`sudo systemctl enable --now atd`<br>
`at -V`<br>
`whatis at`

Downloading script<br>
`wget https://raw.githubusercontent.com/chihen73/P75J900/main/AtOperator/at-example.txt`

Scheduling the script<br>
`sudo rm -rf /tmp/at-example-result.txt`<br>
`at -M -v -f at-example.txt 'now + 1 minute'`

Observe results<br>
`watch -n 1 cat /tmp/at-example-result.txt`

### AtOperator

#### Demo environment setup

Using Cloud Shell<br>
Change to sudo mode<br>
`sudo -i`

Uninstall GO<br>
`which go`<br>
`sudo rm -rvf /usr/local/go/bin/go`

Install GO 1.17.13<br>
`wget https://go.dev/dl/go1.17.13.linux-amd64.tar.gz`<br>
`rm -rf /usr/local/go && tar -C /usr/local -xzf go1.17.13.linux-amd64.tar.gz`<br>
`go version`

Install Kubebuilder
```
version=v3.3.0
os=$(go env GOOS)
arch=$(go env GOARCH)
curl -L -o kubebuilder https://github.com/kubernetes-sigs/kubebuilder/releases/download/${version}/kubebuilder_${os}_${arch}
chmod +x kubebuilder && mv kubebuilder /usr/local/bin/
```
Install tree<br>
`apt update`<br>
`apt install tree`

Logout sudo mode<br>
`exit`

Setting environment variables<br>
`echo export PATH=$PATH:/usr/local/go/bin >>~/.bashrc`<br>
`echo export GOPATH=$PWD/go >>~/.bashrc`<br>
`go env`

#### Building AtOperator

Initialize Kuberbuilder<br>
`mkdir -p $GOPATH/src/example; cd $GOPATH/src/example`<br>
`kubebuilder init --domain d2iq.com`

kubebuilder create a new API

```
kubebuilder create api \
 --group cnat \
 --version v1alpha1 \
 --controller \
 --resource \
 --kind At
```
Copy function codes from Repo
```
curl https://raw.githubusercontent.com/chihen73/P75J900/main/AtOperator/at_types.go > api/v1alpha1/at_types.go
curl https://raw.githubusercontent.com/chihen73/P75J900/main/AtOperator/at_controller.go > controllers/at_controller.go
curl https://raw.githubusercontent.com/chihen73/P75J900/main/AtOperator/main.go > main.go
curl https://raw.githubusercontent.com/chihen73/P75J900/main/AtOperator/Makefile > Makefile
wget https://raw.githubusercontent.com/chihen73/P75J900/main/AtOperator/at-crd-instance.yaml
```
**(In another terminal)** Observing<br>
`watch -n 1 kubectl get ats,pods -o wide`

Install CRDs into the K8s cluster<br>
`make install`

**(In another terminal)** Run a controller from your host<br>
`make run`

Get a server time that will be two minutes from now:<br>
`date -d "$(date --utc +%FT%TZ) + 2 min" +%FT%TZ`

Copy the date and paste it into the quote for the schedule field of at-crd-instance.yaml<br>
`vim at-crd-instance.yaml`

Create a Fresh At Resource<br>
`kubectl apply -f at-crd-instance.yaml`

### Clean-up<br>

Delete GKE cluster!!!<br>
Delete GKE cluster!!!<br>
Delete GKE cluster!!!

---

## Reference

[Kubernetes Patterns, 2nd Edition.](https://www.oreilly.com/library/view/kubernetes-patterns-2nd/9781098131678/) By Bilgin Ibryam, Roland Huss. O'Reilly Media, Inc. March 2023.<br>
[Kubernetes Intermediate in 3 Weeks—with Interactivity](https://learning.oreilly.com/live-events/kubernetes-intermediate-in-3-weekswith-interactivity/0636920056954/), Week 1: Designing with Operators. By Jonathan Johnson. O'Reilly Media, Inc. April 2023.

## Resource

[CNCF Operator White Paper](https://tag-app-delivery.cncf.io/whitepapers/operator/)
