# Kubernetes HA Education
## Rollout Strategies
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy

In a deployment, the `strategy` determines how new Pods are rolled out, most commonly for example when there is a new version of their image.

The most common two strategies are:
- `RollingUpdate`: New Pods are created, and when ready, the old ones are terminated. This is suitable for stateless pods, where old and new versions can run simultaneously. 

This is the default strategy, if not otherwise specified.
 
- `Recreate`: All Pods will be terminated and then the new ones will be created. This will of course cause downtime, until one of the new pods is ready again. This is used where old and new versions of a Pod can not run simultaneously, eg. when they use a RWX volume together and the file structure needs to be modified for the new version.

### Demo for `RollingUpdate` rollout strategy
1. Deploy the app with `RollingUpdate` as the rollout strategy:
```
kubectl apply -f rollout_strategies/rolling.yaml
```

2. Wait for ~2 minutes and check if all pods are ready:
```
kubectl get pods
```

3. Trigger a rollout:
```
oc rollout latest rolling
```

4. Watch how each pod is terminated and recreated one by one:
```
watch -n 1 kubectl get pods -o wide
```

### Demo for `Recreate` rollout strategy
1. Deploy the app with `Recreate` as the rollout strategy:
```
kubectl apply -f rollout_strategies/recreate.yaml
```

2. Wait for ~2 minutes and check if all pods are ready:
```
kubectl get pods
```

3. Trigger a rollout:
```
oc rollout latest recreate
```

4. Watch how every pod is terminated and recreated at the same time:
```
watch -n 1 kubectl get pods -o wide
```

## Probes
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes

To monitor a pods health and if it's ready to accept traffic, the following probes can be used:
* `livenessProbe`: Check if a pod is healthy. If this probe fails, the pod is considered crashed. Kubernetes will terminate and reschedule it.
* `readinessProbe`: Checks if a pod is able to accept traffic. When this probe fails, for example during startup or when it's running and is occupied loading lots of data, the pod is marked as `NotReady` and will not receive traffic from the `Service`. However, Kubernetes will keep it running, waits until the probe succeeds and re-add it to the `Service` to receive traffic again.
* `startupProbe`: Meant for legacy applications, iwhich may take long to start. `LivenessProbe`'s run only after this `startupProbe` succeeds.
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

### Demo
There is a `livenessProbe` declared for the Pods in the `DeploymentConfig` in the file [`rollout_strategies/rolling.yaml`](rollout_strategies/rolling.yaml):
```
        livenessProbe:
          exec:
            command:
            - cat
            - /etc/httpd/tls/localhost.crt
          initialDelaySeconds: 30
          periodSeconds: 1
          successThreshold: 1
          timeoutSeconds: 3
          successThreshold: 1
          timeoutSeconds: 3
```
So this checks for the existance of the file `/etc/httpd/tls/localhost.crt` each second. 

(Be advised, this is just an example for easy demonstration. For Pods with webservices it's better to monitor its health via a HTTP request.)

1. Check the running pods:
```
kubectl get pods
```

2. Delete the file, which the `LivenessProbe` continuously queries, in a running pod:
```
kubectl exec <<POD-NAME>> -- rm -f /etc/httpd/tls/localhost.crt
```

3. Check Pods again, the affected Pod has restarted (see the `RESTARTS` Column):
```
kubectl get pods
```

## Pod Anti-Affinity
https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity

You can use affinity or anti-affinity rules to distribute pods among nodes.

The `DeploymentConfig` declared in [`rollout_strategies/rolling.yaml`](rollout_strategies/rolling.yaml) has such an anti-affinity rule, to start each pod on a seperate node:
```
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - rolling
            topologyKey: "kubernetes.io/hostname"
```

Check the running pods and the nodes they are running on, on how many times each node is used:
```
kubectl get pods -l app=rolling -o wide | tail -n +2 | awk '{ print $7 }' | uniq -c
```

## Pod Disruption Budget (PDB)
https://kubernetes.io/docs/concepts/workloads/pods/disruptions/#pod-disruption-budgets

You can set PDBs to protect Pods, eg. from a Deployment, by limiting how many of them can be unavailable at the same time.

For example if you have a DB or queuing application consisting of 3 pods, where only 1 pod can be down at a time. This way, during maintenance, taking too many pods down simultaniously can be prevented.

---------
TODO: rolling PDB verlinken, testen



## Cleanup Apps
Cleanup all resources of the *recreate*-app:
```
kubectl delete all -l app=recreate
```

Cleanup all resources of the *rolling*-app:
```
kubectl delete all -l app=rolling
```


