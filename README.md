# Kubernetes Gateway API via Cloudflare(d) Tunnels

Manage Kubernetes ingress traffic with Cloudflare(d) Tunnels via the [Gateway API](https://gateway-api.sigs.k8s.io).

## Getting Started

1. Install the helm chart repository: `helm repo add cloudflare-gateway https://geebee.github.io/k8s-gateway-cloudflare`
2. Update the helm repository to fetch the latest release information: `helm repo update cloudflare-gateway`
3. Install the helm chart: `helm install --namespace cloudflare-gateway --create-namespace cloudflare-gateway/cloudflare-gateway cloudflare-gateway`
  - If the Gateway API CRDs are already installed, or you want to skip installing them for a different reason, pass the `--skip-crds` flag to the `helm install` command above
4. [Find your Cloudflare account ID](https://developers.cloudflare.com/fundamentals/setup/find-account-and-zone-ids/)
5. [Create a Cloudflare API token](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/) with the Account.Cloudflare Tunnel and DNS.Zone permissions
6. Use them to create a Secret: `kubectl create secret -n cloudflare-gateway generic cloudflare-credentials --from-literal=ACCOUNT_ID=your-account-id --from-literal=TOKEN=your-token`
7. Create a file containing your GatewayClass, then apply it with `kubectl apply -f gateway-class.yaml`:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: cloudflare
spec:
  controllerName: github.com/geebee/k8s-gateway-cloudflare
  parametersRef:
    group: ""
    kind: Secret
    namespace: cloudflare-gateway
    name: cloudflare-credentials
```

7. [Create Gateways and HTTPRoutes](https://gateway-api.sigs.k8s.io/guides/http-routing/) to start managing traffic! For example:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: cloudflare-gateway
  namespace: default
spec:
  gatewayClassName: cloudflare
  listeners:
  - protocol: HTTP
    port: 80
    name: http
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-route
  namespace: default
spec:
  parentRefs:
  - name: cloudflare-gateway
    namespace: default
  hostnames:
  - example.com
  rules:
  - backendRefs:
    - name: example-service
      port: 80
```

## Features

The complete v1 Core spec is not yet supported, as some features (eg; header-based routing) aren't available with Tunnels.

The following features are supported:

  - `HTTPRoute` hostname and path matching
  - `HTTPRoute` Service backendRefs without filtering or weighting
  - `Gateway` gatewayClassName and listeners only
  - `GatewayClass` Core fields

<!-- * HTTPRoute Gateway parentRefs, without sectionName
* HTTPRoute hostnames, but not listener filtering or precedence
* HTTPRoute rule path match only
* HTTPRoute backendRefs without filtering or weighting
* Gateway gatewayClassName, listeners aren't validated
* GatewayClass Core fields -->
