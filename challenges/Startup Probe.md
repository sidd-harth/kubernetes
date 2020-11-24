# Startup Probe

A Probe is a diagnostic performed periodically by the kubelet on a Container. 

The kubelet can optionally perform and react to three kinds of probes on running containers. The most common and widely known `livenessProbe` and `redinessProbe`.

**Startup probes** are similar to readiness probes but only executed at startup.
They are optimized for slow starting containers or applications.
We can configure `failureThreshold` and `periodSeconds` 
```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 40
  periodSeconds: 5

livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 1
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  failureThreshold: 1
  periodSeconds: 10
```
With this config the application will have a maximum of `200 seconds (40*5)` to finish its startup. Once the startup probe has succeeded, the `liveness probe` takes over.

If the `startup probe` never succeeds, the container is killed after 200s and subject to the pod's `restartPolicy`