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
2. oc create -f yamls/serving.yaml
