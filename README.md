# grpc-gateway-internal-example
super simple use of a gRPC service behind a managed, internal ALB on Google Cloud

Prerequisites:

- GKE cluster with the [Gateway controller](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/deploying-gateways#enable-gateway) enabled 
- the GKE cluster nodes must have the ability to call to an external, Google-managed Artifact Registry (used to pull the image used for the demo app)
- because the load balancer created will be a regional, internal application load balancer, the VPC must have a [proxy-only subnet](https://docs.cloud.google.com/load-balancing/docs/proxy-only-subnets) created 
- have some VM (or workstation) within your dev environment that has [grpcurl](https://github.com/fullstorydev/grpcurl) installed that has connectivity to the node subnet used by the GKE cluster (the VIP used by the load balancer will, by default, pull from the node range)

### setup

```
# assuming you have the proper kube context selected

# create namespaces
kubectl apply -f k8s-namespaces






```