
# Supplementary Services

- [Misc](#misc)
  - [Load Testing](#load-testing)
    - [Fortio](#fortio)
    - [K6 Operator](#k6-operator)
  - [Reloader by Stakater](#reloader-by-stakater)

# Misc

## Load Testing
If you need to put some workload on your apps use [https://github.com/fortio/fortio](Fortio) or [K6 operator](https://github.com/grafana/k6-operator) deployed inside cluster.

### Fortio
Use Fortio if you need to create load on one of services. You can use service FQDN with port to access deployment (e.g. `http://podinfo.apps-dev.svc.cluster.local:9898/`)

### K6 Operator
[K6 operator](https://github.com/grafana/k6-operator) is used to create more inteligent distributed load tests. You need to create a k6 type resourse and a configmap with test configuration written in JS.

Example of ConfigMap:
```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: k6-test-output
  namespace: load-testing
data:
  test.js: |
    import http from 'k6/http';
    import { Rate } from 'k6/metrics';
    import { check, sleep } from 'k6';
    const failRate = new Rate('failed_requests');
    export const options = {
      scenarios: {
        constant_request_rate: {
          executor: 'constant-arrival-rate',
          rate: 1000,
          timeUnit: '1s', // 1000 iterations per second, i.e. 1000 RPS
          duration: '30s',
          preAllocatedVUs: 100, // how large the initial pool of VUs would be
          maxVUs: 200, // if the preAllocatedVUs are not enough, we can initialize more
        },
      },
    };
    export default function () {
      const result = http.get('http://podinfo.apps-dev.svc.cluster.local:9898/');
      check(result, {
        'http response status code is 200': result.status === 200,
      });
      failRate.add(result.status !== 200);
    }
```

Example of k6:
```
---
apiVersion: k6.io/v1alpha1
kind: K6
metadata:
  name: k6-podinfo
  namespace: load-testing
spec:
  parallelism: 4
  script:
    configMap:
      name: k6-test
      file: test.js
```

You can configure tolerations and affinity for k6 jobs on the runner level in spec.