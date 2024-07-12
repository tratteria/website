---
Title: "Deploying Tratteria Service"
weight: 2
toc: true
---

[Tratteria Service](https://github.com/tratteria/tratteria) is an open source Transaction Tokens (TraTs) Service. It is responsible for generating TraTs. 

[Tconfigd](/docs/installation/installing-tconfigd) runs in its own dedicated namespace and is cluster-specific. But, the Tratteria service operates in the application namespace. We must deploy different Tratteria services for different application environments or namespaces.

Please follow the instructions from [Tratteria's deployment readme](https://github.com/tratteria/tratteria/tree/main/kubernetes) to deploy Tratteria service.

After successfully deploying Tratteria service, you should be able to see running Tratteria service pods in the application namespace. For example, below is the running Tratteria pods from the [Tratteria example application's](https://github.com/tratteria/example-application) dev environment. 

```bash
kubectl get pod -n alpha-stocks-dev | grep tratteria
```

Output:

```bash
tratteria-7b89b4f498-6kgd2   1/1     Running   0          22m
tratteria-7b89b4f498-hgxct   1/1     Running   0          22m
tratteria-7b89b4f498-tlrqd   1/1     Running   0          22m
```


Next, proceed to [Setting up Tratteria agents](/docs/installation/setting-up-tratteria-agents), a sidecar for verifying Transaction Tokens (TraTs).

