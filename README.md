# istio-demo-project
Learning Istio n/w from this project

Here's a breakdown of each file and then a visual diagram of the full traffic flow.
frontend.yaml — Defines the frontend Service (port 80) and a Deployment running nginx. This is the user-facing app that external traffic lands on first inside the cluster.

frontend-gateway.yaml — Two Istio resources working together. The Gateway binds to the Istio ingress gateway pod and opens port 80 for all hosts (*). The VirtualService then ties that gateway to the frontend service, routing all / traffic to it. This is the cluster's front door.

api-gateway.yaml — An internal Service + Deployment running hashicorp/http-echo. It simulates a backend API gateway that would call downstream services like orders-service. Exposed only inside the cluster on port 8080.

orders-service.yaml — The core backend service. One Kubernetes Service pointing to two Deployments: orders-service-v1 (2 replicas, using kennethreitz/httpbin) and orders-service-v2 (1 replica). Both share the app: orders-service label but differ on the version label, which is how Istio distinguishes them.

orders-destrule.yaml — A DestinationRule that teaches Istio about the two subsets (v1 and v2) of orders-service by matching on the version label. Also sets ROUND_ROBIN as the load balancing policy within each subset.

orders-vs-canary.yaml — A VirtualService that controls the traffic split between v1 and v2. Right now it's configured with v1: 0% and v2: 100%, meaning all traffic goes to v2. This is a canary setup — you'd typically start at v1: 90 / v2: 10 and gradually shift weight as v2 proves stable.

peerauth-strict.yaml — A PeerAuthentication policy set to STRICT mTLS for the entire microservices namespace. This means every pod-to-pod connection must use mutual TLS — no plaintext traffic is allowed between services. Istio sidecars handle the cert exchange automatically.