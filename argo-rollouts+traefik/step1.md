### Ingress Resources

Deploy In-tree Kubernetes Gateway Api CRDs:

```plain
kubectl apply -f 00-crd-gatewayapi-alpha+beta.yaml
```{{exec}}

Deploy Traefik CRDs:

```plain
kubectl apply -f 01-crd-traefik.yaml
```{{exec}}

Deploy Gateway Api Conformant Traefik Gateway:

```plain
kubectl apply -f 02-gateway-traefik.yaml
```{{exec}}

### Application Deployments

Deploy Argo Rollouts Progressive Delivery Controller:

```plain
kubectl apply -f 03-deployment-argo-rollouts.yaml
```{{exec}}

Deploy Demo Application:

```plain
kubectl apply -f 04-deployment-rollouts-demo.yaml
```{{exec}}

Check current application state (Pre-Canary):

```plain
clear; kubectl get all,traefikservice -n rollouts-demo
```{{exec}}

> Notice: `Kind: Service` or `Kind: TraefikService` are pre-created and staged, however, no pods are running

Deploy `kind: Rollout`

```plain
kubectl apply -f 05-canary-rollouts-demo.yaml
```{{exec}}

### Testing and Monitoring

Check application state (Post-Canary):

```plain
while ! kubectl wait --for=condition=ready pod -n rollouts-demo -l app=rollouts-demo --timeout 1s; do sleep 1; done;
clear; kubectl get all,traefikservice,rollouts -n rollouts-demo
```{{exec}}

Access the application using this link (open in new window for best results):

[Demo Application Link]({{TRAFFIC_HOST1_30080}})

Modify the demo app image to trigger a successful canary deployment

> The demo is set to rollout the new version and stop at 10% and waits for the user to manually promote via the Argo Rollouts dashboard

```plain
kubectl -n rollouts-demo set image deployment/rollouts-demo rolloutd=argoproj/rollouts-demo:green
```{{exec}}

View Argo Rollouts Dashboard and Manuall Promote Canary:

[Argo Rollouts Dashboard]({{TRAFFIC_HOST1_30081}})

Select the namespace, click the canary, click promote!

View Argo Rollout Logs

```plain
kubectl logs -n argo-rollouts -l app.kubernetes.io/name=argo-rollouts
```{{exec}}

Extra:

Try deploying a bad image, view the analysis to see failed response codes (non-200)

```plain
kubectl -n rollouts-demo set image deployment/rollouts-demo rolloutd=argoproj/rollouts-demo:bad-blue
```{{exec}}

done;
