# kubecon24-kepler-knative
Repo with assets to reproduce the talk

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

* crc start --cpus 7

## Install `kn` cli
   *  `wget https://mirror.openshift.com/pub/openshift-v4/clients/serverless/latest/kn-linux-amd64.tar.gz -O my-kn.tar.gz`
   *  `tar -xf my-kn.tar.gz`
   *  `sudo mv kn-linux-amd64 /usr/local/bin/kn`

## Installing the demo
1. Install serverless, power monitoring and opentelemetry operators by  `kubectl apply -f yamls/operators.yaml`

2. Install user workload monitoring (prometheus), serving and kepler instances by:
   * `kubectl apply -f yamls/instances.yaml`

> Check: `oc get knativeserving.operator.knative.dev/knative-serving -n knative-serving --template='{{range .status.conditions}}{{printf "%s=%s\n" .type .status}}{{end}}'`

3. Deploy grafana
   * Simply run [deploy-grafana.sh](https://github.com/sustainable-computing-io/kepler-operator/blob/v1alpha1/hack/dashboard/openshift/deploy-grafana.sh)

4. Install istio with istioctl:
   * `curl -L https://istio.io/downloadIstio | sh -`
   * `istioctl install --set profile=openshift`

6. Create namespaces and enable auto-injection
     * `kubectl create ns serverless-ns`
     * `kubectl create ns serverfull-ns`
     * `kubectl label namespace serverfull-ns serverless-ns istio-injection=enabled --overwrite`
     * `kubectl get namespace -L istio-injection`

7. Install hermes
   * `helm repo add hermes-charts https://jgomezselles.github.io/hermes-charts`
   * `helm repo update charts/kn-hermes`
   * Make sure to check/delete previous runs/namespaces: `kubectl get ns serverfull-ns serverless-ns`
   * Install serverfull instance on its own namespace:
     * `helm install serverfull -n  serverfull-ns charts/kn-hermes/ -f charts/kn-hermes/serverfull_values.yaml`
   * Install serverless instance on its own namespace:
     * `helm install serverless -n  serverless-ns charts/kn-hermes/ -f charts/kn-hermes/serverless_values.yaml --set global.hermes.endpoint="serverless-mock.serverless-ns.svc.cluster.local"`

## Deleting installation
* `helm delete -n serverfull-ns serverfull`
* `helm delete -n serverless-ns serverless`
* `istioctl uninstall --purge`
* `kubectl delete ns istio-system serverfull-ns serverless-ns`
* Delete grafana
* `oc delete -f yamls/serving.yaml`
* `oc delete -f yamls/serverless-operator.yaml`

# Other useful info

## Launch traffic with h2load
  * `h2load http://serverless-mock.hermes.svc.cluster.local:80/url/example/path/hello -m1 -n9000 -c3`

## Queries:
   * `avg(rate(hermes_requests_sent_total{id="post1"}[1m]))`
   * `kepler:kepler:container_joules_total:consumed:24h:by_ns{container_namespace=~"$namespace"} * $watt_per_second_to_kWh`

## Building mock image
   * `docker build -f server-mock/docker/Dockerfile . -t ghcr.io/jgomezselles/kubecon24/server-mock:0.0.1 --progress plain --no-cache`

## Next
* Find the best way to deploy the mock with a LoadBalancer in http/2
* Create dashboard
* See if we can save prometheus metrics to just reuse them
* Change commands to be kubectl
* Change Kepler to community version
* Record the demo

## To monitor
* Serverless:
  * openshift-serverless
  * knative-serving
  * knative-serving-ingress
  * operator
  * Everything in the data path
  * Gateway and pods
  * Keep in mind that using local svc should be fine!

## Useful docs
* https://knative.dev/docs/serving/autoscaling/autoscale-go/ 
* https://knative.dev/docs/serving/load-balancing/target-burst-capacity/ 
  * with this you can enforce that Activator is always in path (if you set the value to “-1”). This way it should work that it select different backends even with only one connection.
  * again in the oc get sks you can see if the Activator is in path (mode: Proxy) or not (mode: Serve)
* Export data https://docs.openshift.com/container-platform/4.15/monitoring/configuring-the-monitoring-stack.html#configuring_remote_write_storage_configuring-the-monitoring-stack 
*  Serverless official [doc](https://docs.openshift.com/serverless/1.31/install/install-serverless-operator.html)
*  Power monitoring docs [instructions](https://docs.openshift.com/container-platform/4.14/observability/power_monitoring/installing-power-monitoring.html)


## Install istio with helm (failed for some issue with the apiserver probably):
   * Following: https://istio.io/latest/docs/setup/install/helm/
   * `helm repo add istio https://istio-release.storage.googleapis.com/charts`
   * `kubectl create namespace istio-system`
   * `helm install istio-base istio/base -n istio-system --set defaultRevision=default`
   * Check `helm ls -n istio-system` is in STATUS `deployed`
   * `helm install istiod istio/istiod -n istio-system --wait`
   * Again, check `helm ls -n istio-system` is in STATUS `deployed`