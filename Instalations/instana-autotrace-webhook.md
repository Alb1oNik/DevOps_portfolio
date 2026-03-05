# Installing instana-autotrace-webhook on OpenShift

Official documentation: https://www.ibm.com/docs/en/instana-observability/1.0.312?topic=webhook-installing-autotrace

---

## Step 1: Check Existing Webhooks

Verify whether any Instana webhooks are already installed in the cluster:

```bash
oc get mutatingwebhookconfigurations | grep instana
```

---

## Step 2: Create Namespace and Install via Helm

Before running the command, configure the flags:

- **Language flags** — set `true` for languages you want to instrument, `false` to skip
- **`webhook.imagePullCredentials.password`** — your Instana download key (found in your Instana installation project)

```bash
helm install --create-namespace --namespace instana-autotrace-webhook instana-autotrace-webhook \
  --repo https://agents.instana.io/helm instana-autotrace-webhook \
  --set webhook.imagePullCredentials.password=<download_key> \
  --set autotrace.instrumentation.manual.ibmmq=false \
  --set autotrace.instrumentation.manual.ruby=false \
  --set autotrace.instrumentation.manual.ibmace=false \
  --set autotrace.instrumentation.manual.netcore=true \
  --set autotrace.instrumentation.manual.nginx=false \
  --set autotrace.instrumentation.manual.nodejs=true \
  --set autotrace.instrumentation.manual.python=true \
  --set autotrace.opt_in=true \
  --set openshift.enabled=true
```

The webhook will automatically instrument the following runtimes via `LD_PRELOAD`:

| Runtime   | Flag                                              |
|-----------|---------------------------------------------------|
| Node.js   | `autotrace.instrumentation.manual.nodejs=true`    |
| .NET Core | `autotrace.instrumentation.manual.netcore=true`   |
| Python    | `autotrace.instrumentation.manual.python=true`    | 
| Ruby      | `autotrace.instrumentation.manual.ruby=false`     | 
| NGINX     | `autotrace.instrumentation.manual.nginx=false`    | 
| IBM MQ    | `autotrace.instrumentation.manual.ibmmq=false`    | 
| IBM ACE   | `autotrace.instrumentation.manual.ibmace=false`   | 

---

## Step 3: Label the Target Namespace

Apply the Instana autotrace label to the namespace you want to instrument:

```bash
oc label namespace <project_name> instana-autotrace=true
```

---

## Step 4: Reload the Application

Restart the application pods in the labeled namespace to apply instrumentation:

```bash
oc rollout restart deployment -n <project_name>
```

> A pod restart is required — the webhook injects instrumentation at pod startup via init containers.