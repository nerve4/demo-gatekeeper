# Demo-Gatekeeper

## Summary

Demo and play regarding Open-Policy Agent and Gatekeeper. OPA/Gatekeeper is 
good for an extra but necessary reinforcment to manage our environment and our 
"little pets". 


## Gatekeeper Deployment

```
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml
kubectl get po -n gatekeeper-system
```
and the output something like this:
```
NAME                                             READY   STATUS    RESTARTS      AGE
gatekeeper-audit-5bbbd695bb-48qxr                1/1     Running   1 (32s ago)   83s
gatekeeper-controller-manager-544c6855d7-gb89p   1/1     Running   0             83s
gatekeeper-controller-manager-544c6855d7-q9hkv   1/1     Running   0             83s
gatekeeper-controller-manager-544c6855d7-s95vl   1/1     Running   0             83s
```


## The 2 example scenario

In this demo, will have a look only 2 example.

### Block NodePort Service

Deploy `ConstraintTemplate` first:
```
cd block-nodeport-services
kubectl apply -f template.yaml
```
then the Rule:
```
kubectl apply -f block-node-port.yaml
```
and try to deploy the example service:
```
kubectl apply -f example-app.yaml

deployment.apps/demo-opa created
ingress.networking.k8s.io/demo-opa created
configmap/nginx-configuration created
Error from server (Forbidden): error when creating "example-app.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [block-node-port] User is not allowed to create service of type NodePort
```

As you see, policy is working. Change the service type to `LoadBalancer` then try to implement again.
```
kubectl get deploy

NAME       READY   UP-TO-DATE   AVAILABLE   AGE
demo-opa   3/3     3            3           66s


kubectl get pods

NAME                        READY   STATUS    RESTARTS   AGE
demo-opa-67644b5b9b-6st77   1/1     Running   0          42s
demo-opa-67644b5b9b-8s87v   1/1     Running   0          42s
demo-opa-67644b5b9b-qqws4   1/1     Running   0          42s


kubectl get ingress

NAME       CLASS   HOSTS       ADDRESS                     PORTS   AGE
demo-opa   nginx   demo.info   <YOUR-EXTERNAL-IP>          80      12s
```
and nginx webservice should be available under `http://demo.info`.

Delete the service with `kubectl delete -f example-app.yaml`.

### Setup ContainerLimit Must

Deploy `ConstraintTemplate` first:
```
cd containerlimits/
kubectl apply -f template.yaml 

constrainttemplate.templates.gatekeeper.sh/k8scontainerlimits created
```
then the Rule:
```
kubectl apply -f container-must-have-limits.yaml
```
Finally, apply the example app deployment:
```
kubectl apply -f example-app.yaml

deployment.apps/demo-opa created
service/demo-opa created
ingress.networking.k8s.io/demo-opa created
configmap/nginx-configuration created
```

All looks normal but...
```
kubectl get deploy

NAME       READY   UP-TO-DATE   AVAILABLE   AGE
demo-opa   0/3     0            0           63s
```
and no pods:
```
kubectl get pods

No resources found in default namespace.
```

Of course after limit fix, pods are coming up and will be fine.


## More Info

  - [GateKeeper Docs](https://open-policy-agent.github.io/gatekeeper/website/docs/)
  - [OPA Git](https://github.com/open-policy-agent)
  - [Rego](https://www.openpolicyagent.org/docs/latest/policy-language/)