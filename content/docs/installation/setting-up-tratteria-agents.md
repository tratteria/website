---
Title: "Setting Up Tratteria Agents"
weight: 3
toc: true
---

[Tratteria Agents](https://github.com/tratteria/tratteria-agent) are sidecar container agents that verify TraTs in microservices.

Tratteria agents are injected into microservices pods to verify TraTs. To integrate Tratteria agents into microservices, follow the instructions from [Tratteria Agent's readme](https://github.com/tratteria/tratteria-agent/blob/main/README.md).

After successfully integrating Tratteria Agents, you should be able to see running Tratteria agent sidecars alongside your application microservices containers. For example, below are the running Order service pods from the [Tratteria example application's](https://github.com/tratteria/example-application) dev environment.

```bash
kubectl get pod -n alpha-stocks-dev | grep order
```

Output:

```bash
order-556d94cfdc-gs9ww       2/2     Running   0          63m
order-556d94cfdc-hjfwd       2/2     Running   0          63m
```

Each Order service replica has two containers. The second container is the Tratteria agent sidecar container.

Next, proceed to writing Tratteria Kubernetes resources for specifying how to generate and verify TraTs for your application APIs: [Generating and Verifying TraTs](/docs/generating-and-verifying-trats).

<br>

For a practical example of installing Tratteria on a microservice application, refer to the [Tratteria example application](https://github.com/tratteria/example-application).