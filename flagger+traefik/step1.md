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

Deploy Flagger and Load Testing Deployments:

```plain
kubectl apply -f 03-deployment-loadtest.yaml
kubectl apply -f 04-deployment-flagger.yaml
```{{exec}}

Deploy Demo Application:

```plain
kubectl apply -f 05-deployment-demoapp.yaml
```{{exec}}

Check current application (Pre-Canary):
> Notice how there are no `Kind: Service` or `Kind: TraefikService`:

```plain
clear; kubectl get all,traefikservice -n canary-demo
```{{exec}}

Deploy `kind: Canary` to expose the application to the outside world:

```plain
kubectl apply -f 06-canary-demoapp.yaml
```{{exec}}

Check current application (Post-Canary):
> Notice `Kind: Service` and `Kind: TraefikService`:

```plain
while ! kubectl wait -n canary-demo replicaset --for=jsonpath='{.status.readyReplicas}'=2 -l app=canary-demo-primary; do sleep 1; done
clear,kubectl get all,traefikservice -n canary-demo
```{{exec}}

Access the application using this link (open in new window for best results):
[demo application link]({{TRAFFIC_HOST1_30080}})

Modify the demo app image to trigger a successful canary deployment

```plain
kubectl -n canary-demo set image deployment/canary-demo rolloutd=argoproj/rollouts-demo:green
```{{exec}}

> The demo is set to rollout the new version incrementally and should fully cut over once all tests are successful

View the rollout in real-time by viewing the percentage of traffic routing to both the primary and canary services (ctrl-C to escape):

```plain
watch 'kubectl get po -n canary-demo ; kubectl get traefikservice -n canary-demo  -oyaml'
```{{exec}}

Now let's modify the demo app with a bad image, to trigger an automated rollback

```plain
kubectl -n canary-demo set image deployment/canary-demo rolloutd=argoproj/rollouts-demo:bad-purple
```{{exec}}

This time we'll watch the Flagger controller logs

```plain
kubectl logs -n flagger -l app.kubernetes.io/name=flagger -f
```{{exec}}

done;
