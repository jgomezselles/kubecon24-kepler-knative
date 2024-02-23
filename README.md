# kubecon24-kepler-knative
Repo with assets to reproduce the talk

## Configuring OpenShift local
* crc setup
* crc start --cpus 7

## Install `kn` cli
   *  `wget https://mirror.openshift.com/pub/openshift-v4/clients/serverless/latest/kn-linux-amd64.tar.gz -O my-kn.tar.gz`
   *  `tar -xf my-kn.tar.gz`
   *  `sudo mv kn-linux-amd64 /usr/local/bin/kn`

## Install serverless
1. Install the operator by following the official [doc](https://docs.openshift.com/serverless/1.31/install/install-serverless-operator.html)
   * `oc create -f yamls/serverless-operator.yaml`
  
2. Create Serving
  * `oc project openshift-serverless`
  * `oc create -f yamls/serving.yaml`
  * Check: `oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`

3. Service
   * `oc create project mock-ns`
   * `kn service create mock --namespace mock-ns --image ghcr.io/jgomezselles/kubecon24/server-mock:0.0.1 --concurrency-limit 4 --force`
   * Need to attack mock-hermes.apps-crc.testing:80

4. Create user-workload-monitoring (Prometheus)
   * `oc create -f yamls/cluster-monitoring-config.yaml`

5. Install hermes
   * `helm install hermes -n mock-ns charts/hermes-job/`

6. Install OTel
   * `oc create -f otel-collector/`

## Building mock image
   * `docker build -f server-mock/docker/Dockerfile . -t ghcr.io/jgomezselles/kubecon24/server-mock:0.0.1 --progress plain --no-cache`

## Deleting installation
* `kn service delete mock`
* `oc delete -f yamls/serving.yaml`
* `oc delete -f yamls/serverless-operator.yaml`

## Doubts
* Internal endpoint?

## Next
* Helm chart for hermes -> https://github.com/helm/chart-releaser-action