# kubernetes-ha-edu


## Deploy Apps
For app with `RollingUpdate` rollout strategy:
```
oc apply -f rollout_strategies/rolling.yaml
```

For app with `Recreate` rollout strategy:
```
oc apply -f rollout_strategies/recreate.yaml
```

## Test Liveness Probe (in the Rolling-App)
Check running pods:
```
oc get pods
```

Delete a running pod:
```
oc exec <<POD-NAME>> -- rm -f /etc/httpd/tls/localhost.crt
```

Check Pods again, the affected Pod has restarted:
```
oc get pods
```

## Pod Anti-Affinity
You can use affinity or anti-affinity rules to distribute pods among nodes. More Information in the Kubernetes Documentation: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity

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
oc get pods -l app=rolling -o wide | tail -n +2 | awk '{ print $7 }' | uniq -c
```

## Cleanup Apps
Cleanup all resources of the *recreate*-app:
```
oc delete all -l app=recreate
```

Cleanup all resources of the *rolling*-app:
```
oc delete all -l app=rolling
```
