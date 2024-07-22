---
Title: "Quickstart Guide"
weight: 2
toc: true
---

Welcome to the Tratteria quickstart guide. This tutorial will guide you through setting up the sample application and observing Tratteria in action.

## Deployment

Follow the instructions in the [README](https://github.com/tratteria/example-application/blob/main/README.md) of the example application to deploy the application.

After successful deployment, you will see a running `tconfigd` pod in the `tratteria-system` namespace:

```bash
kubectl get pod -n tratteria-system
```

Output:

```bash
NAME                        READY   STATUS    RESTARTS   AGE
tconfigd-7c94fd755b-b5tjh   1/1     Running   0          49s
```

You will also see `Tratteria` service pods in the application namespace, which is `alpha-stocks-dev`:

```bash
kubectl get pod -n alpha-stocks-dev | grep tratteria
```

Output:

```bash
tratteria-7789594688-b8jjq   1/1     Running   0          67s
tratteria-7789594688-g565s   1/1     Running   0          67s
tratteria-7789594688-gjs2r   1/1     Running   0          67s
```

## TraTs

The example application has four APIs, and TraTs for each API can be found [here](https://github.com/tratteria/example-application/tree/main/deploy/alpha-stocks-dev/trats).

These TraTs are applied when the application is deployed. You can list them with the following command:

```bash
kubectl get trats -n alpha-stocks-dev
```

Output:

```bash
NAME                      STATUS   AGE    RETRIES
stock-details-api-trat    DONE     118s   0
stock-holdings-api-trat   DONE     118s   0
stock-search-api-trat     DONE     118s   0
stock-trade-api-trat      DONE     118s   0
```

To check the contents of a specific TraT, use:

```bash
kubectl describe trats/stock-trade-api-trat -n alpha-stocks-dev
```

Output:

```bash
Name:         stock-trade-api-trat
Namespace:    alpha-stocks-dev
Labels:       <none>
Annotations:  <none>
API Version:  tratteria.io/v1alpha1
Kind:         TraT
Metadata:
  Creation Timestamp:  2024-07-22T20:36:14Z
  Generation:          1
  Resource Version:    6490948
  UID:                 d7d8c768-7598-4d02-8e4f-e828dff8d0c7
Spec:
  Azd Mapping:
    Action:
      Required:  true
      Value:     ${body.orderType}
    Quantity:
      Required:  true
      Value:     ${body.quantity}
    Stock Id:
      Required:  true
      Value:     ${body.stockId}
  Endpoint:      /api/order
  Method:        POST
  Purp:          stock-trade
  Services:
    Name:      order
    Endpoint:  /internal/stocks
    Name:      stocks
Status:
  Generation Applied:    true
  Retries:               0
  Status:                DONE
  Verification Applied:  true
Events:
  Type    Reason                                     Age    From             Message
  ----    ------                                     ----   ----             -------
  Normal  verification application stage successful  2m39s  trat-controller  verification application stage completed successfully
  Normal  generation application stage successful    2m39s  trat-controller  generation application stage completed successfully
```

## Observing TraTs in Action

To see TraTs in action, you can use the application UI, which interactively shows the generated TraTs for each invoked external API.

1. Launch the frontend client application as per the example application [README](https://github.com/tratteria/example-application/blob/main/README.md).

2. Enter [http://localhost:4200/](http://localhost:4200/) in your browser.

3. You will see a Dex authentication page similar to the one shown below:

<img src="/img/docs/introduction/ui-dex-auth-page.png" alt="What Is a TraT" class="doc-image">

4. Authenticate with any user from [here](https://github.com/tratteria/example-application/blob/main/deploy/alpha-stocks-dev/configs/dex-config.yaml) with the default password of `password123`. For example, you can use the username `sherlock@detect.com` with password `password123`.

5. After authentication, as you navigate through the application, you will see the generated TraTs for the invoked external APIs. Below is a sample page:

<img src="/img/docs/introduction/ui-trat-page.png" alt="What Is a TraT" class="doc-image">

## Next Steps

To set up TraTs in your own microservices application, proceed to [Installing Tratteria](/docs/installation).