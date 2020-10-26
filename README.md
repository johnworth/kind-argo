# Deploying Argo Projects in Kind

This is just a playground to figure out how to get stuff working. No reason to keep it private, but also probably not useful to anyone else.

# Instructions

`kind create cluster`
`kubectl create ns argo`

## Install Argo Workflows

Based on info in their quickstart at https://argoproj.github.io/argo/quick-start/.

Install Argo Workflows with the `containerRuntimeExecutor: k8sapi` set for the ` workflow-controller-configmap` ConfigMap. The `quick-start-postgres.yaml` file in this repository already has that set.

```bash
    kubectl apply -n argo -f quick-start-postgres.yaml
```

In another terminal tab forward the workflow port.

```bash
    kubectl -n argo port-forward deployment/argo-server 2746:2746
```

Install the Argo cmd-line tool using the instructions on the [Argo Releases](https://github.com/argoproj/argo/releases) page.

Test with Argo Workflow's Hello World. Directly from their quick start:

```bash
    argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo/master/examples/hello-world.yaml
    argo list -n argo
    argo get -n argo @latest
    argo logs -n argo @latest
```

## Install Argo Events

Based on info in their [Installation](https://argoproj.github.io/argo-events/installation/) and [Getting Started](https://argoproj.github.io/argo-events/quick_start/) guides.

```bash
    kubectl create ns argo-events
```

Install the Custom Resource definitions and base services.

```bash
    kubectl apply -n argo-events -f argoevents.yaml
```

Install the event-bus.

```bash
    kubectl apply -n argo-events -f eventbus-native.yaml
```

Install the webhook event source.

```bash
    kubectl apply -n argo-events -f webhook-source.yaml
```

Install the webhook event sensor.

```bash
    kubectl apply -n argo-events -f webhook-sensor.yaml
```

In another tab, port forward the event source service (instructions here vary a bit from the quick start guide on Argo's site).

```bash
    kubectl -n argo-events port-forward svc/webhook-eventsource-svc 12000:12000
```

Send a post request to the event source.

```bash
    curl -d '{"message":"this is my first webhook"}' -H "Content-Type: application/json" -X POST http://localhost:12000/example
```

Check that the Argo workflow was triggered (though not necessarily run):

```bash
    kubectl -n argo-events get workflows | grep webhook
```
