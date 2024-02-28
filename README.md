# kubecon24-kepler-knative
Repo with assets to reproduce the talk

## Configuring OpenShift local
* crc setup
* crc start --cpus 7

## Install `kn` cli
   *  `wget https://mirror.openshift.com/pub/openshift-v4/clients/serverless/latest/kn-linux-amd64.tar.gz -O my-kn.tar.gz`
   *  `tar -xf my-kn.tar.gz`
   *  `sudo mv kn-linux-amd64 /usr/local/bin/kn`

## Installing the demo
1. Install serverless, power monitoring and opentelemetry operators by  `kubectl apply -f yamls/operators.yaml`

> Serverless official [doc](https://docs.openshift.com/serverless/1.31/install/install-serverless-operator.html)
> Power monitoring docs [instructions](https://docs.openshift.com/container-platform/4.14/observability/power_monitoring/installing-power-monitoring.html)
  
2. Install user workload monitoring (prometheus), serving and kepler instances by:
   * `kubectl apply -f yamls/instances.yaml`

> Check: `oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`

3. Create the KN Service on its own namespace
   * `kubectl create ns serverless-ns`
   * `kn service create mock --namespace serverless-ns --image ghcr.io/jgomezselles/kubecon24/server-mock:0.0.1 --concurrency-limit 4 --force`
   * Need to attack mock-hermes.apps-crc.testing:80

5. Deploy grafana
   * Simply run [deploy-grafana.sh](https://github.com/sustainable-computing-io/kepler-operator/blob/v1alpha1/hack/dashboard/openshift/deploy-grafana.sh)

6. Install hermes
   * `helm repo add hermes-charts https://jgomezselles.github.io/hermes-charts`
   * `helm install kn-hermes -n serverless-ns charts/kn-hermes/`


# Other useful info

## Queries:
   * `avg(rate(hermes_requests_sent_total{id="post1"}[1m]))`
   * `kepler:kepler:container_joules_total:consumed:24h:by_ns{container_namespace=~"$namespace"} * $watt_per_second_to_kWh`

## Building mock image
   * `docker build -f server-mock/docker/Dockerfile . -t ghcr.io/jgomezselles/kubecon24/server-mock:0.0.1 --progress plain --no-cache`

## Deleting installation
* `kn service delete mock`
* `oc delete -f yamls/serving.yaml`
* `oc delete -f yamls/serverless-operator.yaml`

## Doubts
* Internal endpoint?

## Next
* Update helm and installations with https://jgomezselles.github.io/hermes-charts/
* Deploy mock as server
* Add OTel collector to helm charts?
* Update namespaces in svc monitor and collector
* Find the best way to deploy the mock with a LoadBalancer in http/2
* Install both at the same time and run
* Create dashboard
* See if we can save prometheus metrics to just reuse them
* Change commands to be kubectl
* Change Kepler to community version
* Record the demo

## Fix
* Fix service monitor in the otel collector
* Add endpoint to hermes to export OTLP metrics