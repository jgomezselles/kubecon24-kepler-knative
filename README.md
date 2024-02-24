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

5. Install power monitoring
   * Follow [instructions](https://docs.openshift.com/container-platform/4.14/observability/power_monitoring/installing-power-monitoring.html)

6. Deploy grafana
   * Simply run [deploy-grafana.sh](https://github.com/sustainable-computing-io/kepler-operator/blob/v1alpha1/hack/dashboard/openshift/deploy-grafana.sh)

7. Install hermes
   * `helm install hermes -n mock-ns charts/hermes-job/`

8. Install OTel
   * `oc create -f otel-collector/`
   * Check by `oc get csv -n openshift-opentelemetry-operator`

9. Create service monitor
   * `oc create -f otel-service-monitor.yaml`

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
* Change to K8s job
* Deploy mock as server
* Find the best way to deploy the mock with a LoadBalancer in http/2

## Fix
* Fix service monitor in the otel collector
* Add endpoint to hermes to export OTLP metrics