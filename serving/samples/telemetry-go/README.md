# Telemetry Sample

This sample runs a simple web server that makes calls to other in-cluster services
and responds to requests with "Hello World!".
The purpose of this sample is to show generating metrics, logs and distributed traces
(see [Logs](../../accessing-logs.md), [Metrics](../../accessing-metrics.md), and [Traces](../../accessing-traces.md) for more information).
This sample also creates a dedicated Prometheus instances rather than using the one
that is installed by default as a showcase of installing dedicated Prometheus instances.

## Prerequisites

1. [Install Knative Serving](https://github.com/knative/docs/blob/master/install/README.md)
2. [Install Knative monitoring component](../../installing-logging-metrics-traces.md)
3. Install [docker](https://www.docker.com/)


## Setup

Build the app container and publish it to your registry of choice:

```shell
REPO="gcr.io/<your-project-here>"

# Build and publish the container, run from the root directory.
docker build \
  --tag "${REPO}/serving/samples/telemetry-go" \
  --file=serving/samples/telemetry-go/Dockerfile .
docker push "${REPO}/serving/samples/telemetry-go"

# Replace the image reference with our published image.
perl -pi -e "s@github.com/knative/docs/serving/samples/telemetry-go@${REPO}/serving/samples/telemetry-go@g" serving/samples/telemetry-go/*.yaml

# Deploy the Knative Serving sample
kubectl apply -f serving/samples/telemetry-go/
```

## Exploring

Once deployed, you can inspect the created resources with `kubectl` commands:

```shell
# This will show the route that we created:
kubectl get route -o yaml

# This will show the configuration that we created:
kubectl get configurations -o yaml

# This will show the Revision that was created by our configuration:
kubectl get revisions -o yaml
```

To access this service via `curl`, we first need to determine its ingress address:
```shell
watch kubectl get svc knative-ingressgateway -n istio-system
NAME                     TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                                      AGE
knative-ingressgateway   LoadBalancer   10.23.247.74   35.203.155.229   80:32380/TCP,443:32390/TCP,32400:32400/TCP   2d
```

Once the `EXTERNAL-IP` gets assigned to the cluster, you can run:

```shell
# Put the Host name into an environment variable.
export SERVICE_HOST=`kubectl get route telemetrysample-route -o jsonpath="{.status.domain}"`

# Put the ingress IP into an environment variable.
export SERVICE_IP=`kubectl get svc knative-ingressgateway -n istio-system -o jsonpath="{.status.loadBalancer.ingress[*].ip}"`

# Curl the ingress IP "as-if" DNS were properly configured.
curl --header "Host:$SERVICE_HOST" http://${SERVICE_IP}
Hello World!
```

Generate some logs to STDOUT and files under `/var/log` in `Json` or plain text formats.

```shell
curl --header "Host:$SERVICE_HOST" http://${SERVICE_IP}/log
Sending logs done.
```

## Accessing logs
You can access to the logs from Kibana UI - see [Logs](../../accessing-logs.md) for more information.

## Accessing per request traces
You can access to per request traces from Zipkin UI - see [Traces](../../accessing-traces.md) for more information.

## Accessing custom metrics
You can see published metrics using Prometheus UI. To access to the UI, forward the Prometheus server to your machine:

```bash
kubectl port-forward $(kubectl get pods --selector=app=prometheus,prometheus=test --output=jsonpath="{.items[0].metadata.name}") 9090
```

Then browse to http://localhost:9090.

## Cleaning up

To clean up the sample service:

```shell
kubectl delete -f serving/samples/telemetrysample-go/
```
