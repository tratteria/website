---
title: "TraTExclusion"
weight: 2
toc: true
---

The TraTExclusion resource is used to specify endpoints of a service that do not require TraTs. This is particularly useful for non-functional APIs, such as health checks and monitoring, or for functional APIs that have not yet been adapted to support TraTs. The configuration is specific to each service.

Below is an example of TraTExclusion for the order service:

```yaml
apiVersion: tratteria.io/v1alpha1
kind: TraTExclusion
metadata:
  name: order-service-tratexcl
  namespace: alpha-stocks-dev
spec:
  service: order
  endpoints:
    - path: "/health"
      method: "GET"
```