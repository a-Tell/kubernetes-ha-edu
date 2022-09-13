# kubernetes-ha-edu


## Deploy Apps
For app with `RollingUpdate` rollout strategy:
```
kubectl apply -f rollout_strategies/rolling.yaml
```

For app with `Recreate` rollout strategy:
```
kubectl apply -f rollout_strategies/recreate.yaml
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

## Cleanup Apps
Cleanup all resources of the *recreate*-app:
```
kubectl delete all -l app=recreate
```

Cleanup all resources of the *rolling*-app:
```
kubectl delete all -l app=rolling
```
