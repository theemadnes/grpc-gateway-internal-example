# grpc-gateway-internal-example
super simple use of a gRPC service behind a managed, internal ALB on Google Cloud

This setup will use *plaintext* both between the client and the load balancer (over port 80) and plaintext (using h2c, configured via the service's `appProtocol` field) between the load balancer and the backend gRPC service. You are welcome to adapt this code to enable TLS termination at the load balancer layer if your situation dictates, but that's up to you. 

The request path in this example is as follows:

`grpcurl client -> load balancer (configured via GKE Gateway API) -> gRPC service`


Prerequisites:

- GKE cluster with the [Gateway controller](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/deploying-gateways#enable-gateway) enabled 
- the GKE cluster nodes must have the ability to call to an external, Google-managed Artifact Registry (used to pull the image used for the demo app)
- because the load balancer created will be a regional, internal application load balancer (using the `gke-l7-rilb` `gatewayClass`), the VPC must have a [proxy-only subnet](https://docs.cloud.google.com/load-balancing/docs/proxy-only-subnets) created 
- have some VM (or workstation) within your dev environment that has [grpcurl](https://github.com/fullstorydev/grpcurl) installed and also has connectivity to the node subnet used by the GKE cluster (the VIP used by the load balancer will, by default, pull from the node range)

### setup

```
# assuming you have the proper kube context selected

# create namespaces
kubectl apply -f k8s-namespaces

# deploy whereami app in gRPC mode, listening on port 9090 on local mode (but load balancer will expose on port 80)
kubectl apply -f whereami

# deploy gateway, httproute, and gRPC healthcheck for whereami (check the namespaces to see where each resource is deployed)
kubectl apply -f gateway-resources

# it will take a few minutes to create the load balancer via the gateway resource
# you can check the status via 
kubectl -n demo-gateway get gateway internal-http

```

### client access
```
# once the ADDRESS field is populated, lets capture it to call it later
# the output will look someting like this
# $ kubectl -n demo-gateway get gateway internal-http
# NAME            CLASS         ADDRESS        PROGRAMMED   AGE
# internal-http   gke-l7-rilb   10.128.0.117   True         112s

# if you're using a workstation with IP connectivity to the range the ALB is using, just capture it to a variable
export GATEWAY_IP=$(kubectl get gateway internal-http -n demo-gateway -o jsonpath='{.status.addresses[0].value}')
echo $GATEWAY_IP

# from a folder with whereami.proto (or navigating to it manually) run grpcurl
grpcurl -plaintext -proto whereami.proto $GATEWAY_IP:80 whereami.Whereami.GetPayload
```

Expected output:

```
$ grpcurl -plaintext -proto whereami.proto $GATEWAY_IP:80 whereami.Whereami.GetPayload
{
  "clusterName": "edge-to-mesh-01",
  "metadata": "grpc-frontend",
  "nodeName": "gk3-edge-to-mesh-01-nap-1c08r7is-873c2a55-taeb",
  "podIp": "10.54.1.165",
  "podName": "whereami-grpc-585d8f95f9-m4b6g",
  "podNameEmoji": "Ⓜ",
  "podNamespace": "demo-whereami",
  "podServiceAccount": "whereami-grpc",
  "projectId": "e2m-private-test-01",
  "timestamp": "2026-01-18T10:41:24",
  "zone": "us-central1-a",
  "gceInstanceId": "4799585712519873663",
  "gceServiceAccount": "e2m-private-test-01.svc.id.goog"
}
```