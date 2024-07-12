---
Title: "Installing Tconfigd"
weight: 1
toc: true
---

[Tconfigd](https://github.com/tratteria/tconfigd) is a central daemon that manages Tratteria configurations. To get started with Tratteria, we first need to install Tconfigd. Please follow the instructions from [Tconfigd's installation readme](https://github.com/tratteria/tconfigd/tree/main/installation) to install Tconfigd.

After successfully installing Tconfigd, you should be able to see a running Tconfigd pod in the tratteria-system namespace.

```bash
kubectl get pod -n tratteria-system
```

Output:

```bash
NAME                        READY   STATUS    RESTARTS   AGE
tconfigd-85c5697c9b-4lx4s   1/1     Running   0          88s
```

Next, proceed to [deploy Tratteria service](/docs/installation/deploying-tratteria), an open-source service for managing Transaction Tokens (TraTs).
