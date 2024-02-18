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

3. Functions
   * Exploring [docs](https://docs.openshift.com/serverless/1.31/functions/serverless-functions-getting-started.html)
   * function created with `kn func create -l go -t http fn`
   * `kn func build --builder=pack --image ghcr.io/jgomezselles/kubecon24-kepler-knative/fn1:0.0.1`
   * Having issue: https://github.com/knative/func/issues/2154

4. Service
   * `kn service create mock --image ttl.sh/jgomezselles/server-mock:0.0.1 --force`
   * Need to attack mock-hermes.apps-crc.testing:80

## Doubts
* Internal endpoint?
* Scale rules

## Next
* Upload mock image
* Helm chart for hermes -> https://github.com/helm/chart-releaser-action