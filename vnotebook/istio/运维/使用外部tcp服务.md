# 使用外部tcp服务
```
$ kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: mongo
spec:
  hosts:
  - my-mongo.tcp.svc
  addresses:
  - $MONGODB_IP/32
  ports:
  - number: $MONGODB_PORT
    name: tcp
    protocol: TCP
  location: MESH_EXTERNAL
  resolution: STATIC
  endpoints:
  - address: $MONGODB_IP
EOF
```

```
$ kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: $EGRESS_GATEWAY_MONGODB_PORT
      name: tcp
      protocol: TCP
    hosts:
    - my-mongo.tcp.svc
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: egressgateway-for-mongo
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
  - name: mongo
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: mongo
spec:
  host: my-mongo.tcp.svc
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: direct-mongo-through-egress-gateway
spec:
  hosts:
  - my-mongo.tcp.svc
  gateways:
  - mesh
  - istio-egressgateway
  tcp:
  - match:
    - gateways:
      - mesh
      destinationSubnets:
      - $MONGODB_IP/32
      port: $MONGODB_PORT
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: mongo
        port:
          number: $EGRESS_GATEWAY_MONGODB_PORT
  - match:
    - gateways:
      - istio-egressgateway
      port: $EGRESS_GATEWAY_MONGODB_PORT
    route:
    - destination:
        host: my-mongo.tcp.svc
        port:
          number: $MONGODB_PORT
      weight: 100
EOF
```