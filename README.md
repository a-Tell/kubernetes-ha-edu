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
oc exec rolling-1-jhdhg -ti -- rm -f /etc/httpd/tls/localhost.crt
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
