# kubecon24-kepler-knative
Repo with assets to reproduce the talk

## Install `kn` cli
   *  `wget https://mirror.openshift.com/pub/openshift-v4/clients/serverless/latest/kn-linux-amd64.tar.gz -O my-kn.tar.gz`
   *  `tar -xf my-kn.tar.gz`
   *  `sudo mv kn-linux-amd64 /usr/local/bin/kn`

## Installing the demo
1. Install serverless, power monitoring and opentelemetry operators by  `kubectl apply -f yamls/operators.yaml`

2. Install user workload monitoring (prometheus), serving, kepler instances and namespaces by:
   * `kubectl apply -f yamls/instances.yaml`

> Check: `oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`

3. Deploy grafana
   * Simply run [deploy-grafana.sh](https://github.com/sustainable-computing-io/kepler-operator/blob/v1alpha1/hack/dashboard/openshift/deploy-grafana.sh)

4. Install istio with istioctl:
   * `curl -L https://istio.io/downloadIstio | sh -`
   * `istioctl install --set profile=openshift --set meshConfig.defaultConfig.holdApplicationUntilProxyStarts=true`

5. Install hermes
   * `helm repo add hermes-charts https://jgomezselles.github.io/hermes-charts`
   * `helm dep update charts/kn-hermes`
   * Install serverfull instance on its own namespace:
     * `helm install serverfull -n  serverfull-ns charts/kn-hermes/ -f charts/kn-hermes/serverfull_values.yaml`
   * Install serverless instance on its own namespace:
     * `helm install serverless -n  serverless-ns charts/kn-hermes/ -f charts/kn-hermes/serverless_values.yaml`
   * Install plain instance (no serverless, no istio):
     * `helm install plain -n plain-ns charts/kn-hermes/ -f charts/kn-hermes/plain_values.yaml`

6. Load `dashboard.json` from this repo to the grafana instance and overwrite (Change UId manually)


## Example number 5 can be run with

```
helm install serverless -n  serverless-ns charts/kn-hermes/ -f charts/kn-hermes/serverless_values.yaml --set mock.image="quay.io/kevindubois/quarked:jvm" --set global.hermes.rate=6 --set global.hermes.cm=get-cm; helm install plain -n plain-ns charts/kn-hermes/ -f charts/kn-hermes/plain_values.yaml  --set mock.image="quay.io/kevindubois/quarked:jvm" --set global.hermes.rate=6 --set global.hermes.cm=get-cm; helm install serverfull -n  serverfull-ns charts/kn-hermes/ -f charts/kn-hermes/serverfull_values.yaml --set mock.image="quay.io/kevindubois/quarked:jvm" --set global.hermes.rate=6 --set global.hermes.cm=get-cm
```

## Deleting installation
* `helm delete -n serverfull-ns serverfull`
* `helm delete -n serverless-ns serverless`
* `helm delete -n plain-ns plain`
* `istioctl uninstall --purge` --> WARNING! This is cluster scoped!
* Delete grafana
* `oc delete -f yamls/instances.yaml`
* `oc delete -f yamls/operators.yaml`

# Other useful info

## Launch traffic with h2load
  * `h2load http://serverless-mock.hermes.svc.cluster.local:80/url/example/path/hello -m1 -n9000 -c3`

## Queries:
   * `avg(rate(hermes_requests_sent_total{id="post1"}[1m]))`
   * `kepler:kepler:container_joules_total:consumed:24h:by_ns{container_namespace=~"$namespace"} * $watt_per_second_to_kWh`

## Building mock image
   * `docker build -f server-mock/docker/Dockerfile . -t ghcr.io/jgomezselles/kubecon24/server-mock:0.0.1 --progress plain --no-cache`

## To monitor
* Serverless:
  * openshift-serverless
  * knative-serving
  * knative-serving-ingress
  * operator

## Useful docs
* https://knative.dev/docs/serving/autoscaling/autoscale-go/ 
* https://knative.dev/docs/serving/load-balancing/target-burst-capacity/ 
  * with this you can enforce that Activator is always in path (if you set the value to “-1”). This way it should work that it select different backends even with only one connection.
  * again in the oc get sks you can see if the Activator is in path (mode: Proxy) or not (mode: Serve)
* Export data https://docs.openshift.com/container-platform/4.15/monitoring/configuring-the-monitoring-stack.html#configuring_remote_write_storage_configuring-the-monitoring-stack 
*  Serverless official [doc](https://docs.openshift.com/serverless/1.31/install/install-serverless-operator.html)
*  Power monitoring docs [instructions](https://docs.openshift.com/container-platform/4.14/observability/power_monitoring/installing-power-monitoring.html)

## Configuring OpenShift local
* crc setup
* crc config set enable-cluster-monitoring true

```
crc config view
- consent-telemetry                     : no
- cpus                                  : 7
- enable-cluster-monitoring             : true
- memory                                : 19000

```