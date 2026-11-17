# Kubernetes Prompt Portfolio

A portfolio of prompts for generating Kubernetes manifests with AI, based on the manifests from [den-vasyliev/go-demo-app/yaml](https://github.com/den-vasyliev/go-demo-app/tree/master/yaml).

All manifests in [`yaml/`](yaml/) were generated with [kubectl-ai](https://github.com/GoogleCloudPlatform/kubectl-ai) (Gemini Flash family models) from the prompts below and validated with `kubectl apply --dry-run=client`.

## Prompts

| NAME | PROMPT | DESCRIPTION | EXAMPLE |
|------|--------|-------------|---------|
| app | You are a Kubernetes expert. Generate a Kubernetes manifest (apiVersion v1) for a single Pod named app with labels app: demo and run: demo. It runs one container named app from image gcr.io/k8s-k3s/demo:v1.0.0 exposing containerPort 8000 named http. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | Basic Pod: one container, labels and a named container port. | [app.yaml](yaml/app.yaml) |
| app-livenessProbe | You are a Kubernetes expert. Generate a Kubernetes manifest (apiVersion v1) for a Pod named app-livenessprobe in namespace demo. It runs one container named app from image gcr.io/smartcity-gl/demo:v1.0.1 exposing containerPort 8000 named http. Add an HTTP livenessProbe on path / and port 8000 with initialDelaySeconds 5, timeoutSeconds 1, periodSeconds 10 and failureThreshold 3. The probe port must match the containerPort. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | Pod with an HTTP liveness probe: kubelet restarts the container after 3 failed checks. Fixes the original's port mismatch (probe on 8000, container on 8080), which would cause an endless restart loop. | [app-livenessProbe.yaml](yaml/app-livenessProbe.yaml) |
| app-readinessProbe | You are a Kubernetes expert. Generate a Kubernetes manifest (apiVersion v1) for a Pod named app-readinessprobe. It runs one container named app from image gcr.io/smartcity-gl/harmonic:v2.0.2 exposing containerPort 8000 named http. Add an HTTP livenessProbe on path / with initialDelaySeconds 5, timeoutSeconds 1, periodSeconds 10 and failureThreshold 3. Add an HTTP readinessProbe on path /ready with initialDelaySeconds 0, periodSeconds 2, failureThreshold 3 and successThreshold 1. Both probes must reference the named port http instead of a number. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | Pod with liveness (restart if hung) and readiness (remove from Service endpoints until ready) probes. Probes reference the named port http, so the port is defined in one place. | [app-readinessProbe.yaml](yaml/app-readinessProbe.yaml) |
| app-volumeMounts | You are a Kubernetes expert. Generate a Kubernetes manifest (apiVersion v1) for a Pod named app-volume. It runs one container named app from image gcr.io/kuar-demo/kuard-amd64:1 exposing containerPort 8080 named http. Add an HTTP livenessProbe on path /healthy with initialDelaySeconds 5, timeoutSeconds 1, periodSeconds 10 and failureThreshold 3, and an HTTP readinessProbe on path /ready with initialDelaySeconds 0, periodSeconds 2, failureThreshold 3 and successThreshold 1; both probes use the named port http. Define a pod volume named data of type hostPath with path /var/lib/app and type DirectoryOrCreate, and mount it into the container at /data. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | Pod with a hostPath volume (spec.volumes) mounted into the container (volumeMounts) at /data. type DirectoryOrCreate makes kubelet create the directory on the node if it is missing. | [app-volumeMounts.yaml](yaml/app-volumeMounts.yaml) |
| app-cronjob | You are a Kubernetes expert. Generate a Kubernetes CronJob manifest using apiVersion batch/v1 (not the removed batch/v1beta1) named app-cronjob that runs every 5 minutes with concurrencyPolicy Forbid. The job template runs one container named hello from image bash with command ["echo", "Hello world"] and restartPolicy OnFailure. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | CronJob that prints "Hello world" every 5 minutes. Migrated from batch/v1beta1 (removed in Kubernetes 1.25) to batch/v1; concurrencyPolicy Forbid prevents overlapping runs. | [app-cronjob.yaml](yaml/app-cronjob.yaml) |
| app-job | You are a Kubernetes expert. Generate a Kubernetes Job manifest (apiVersion batch/v1) named app-job with backoffLimit 0 and ttlSecondsAfterFinished 600. The pod template uses restartPolicy Never and two emptyDir volumes named data-input and data-output. An initContainer named init from image busybox mounts data-input at /data/input and simulates a data download by running /bin/sh -c with a command that writes a sample file /data/input/sample.txt. The main container named processor from image busybox mounts data-input at /data/input and data-output at /data/output and runs /bin/sh -c 'ls -l /data/input'. Add lifecycle hooks to the processor: a postStart exec hook that writes 'Start from the postStart handler' to /data/input/message, and a preStop exec hook that writes 'Stop from the preStop handler' to /data/output/message. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | One-off Job: an initContainer prepares input data, the main container processes it; postStart/preStop lifecycle hooks; no retries (backoffLimit 0); auto-cleanup after 10 minutes (ttlSecondsAfterFinished). The original's GCE persistent disk and private GCS bucket are replaced with emptyDir and a simulated download, so the manifest runs on any cluster. | [app-job.yaml](yaml/app-job.yaml) |
| app-multicontainer | You are a Kubernetes expert. Generate a Kubernetes manifest (apiVersion v1) for a Pod named app-two-containers demonstrating the sidecar pattern. Define an emptyDir volume named html. The first container named nginx uses image nginx, exposes containerPort 80 and mounts the html volume at /usr/share/nginx/html. The second container named content-writer uses image debian, mounts the same html volume at /html and runs command ["/bin/sh", "-c"] with a single argument: a shell loop that appends the output of date to /html/index.html every second. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | Sidecar pattern: two containers share an emptyDir volume mounted at different paths. content-writer appends the date to index.html every second, nginx serves it. Containers get meaningful names instead of 1st/2nd. | [app-multicontainer.yaml](yaml/app-multicontainer.yaml) |
| app-resources | You are a Kubernetes expert. Generate a Kubernetes manifest (apiVersion v1) for a Pod named app-resource. It runs one container named app from image gcr.io/kuar-demo/kuard-amd64:1 exposing containerPort 8080 named http. Add an HTTP livenessProbe on path /healthy with initialDelaySeconds 5, timeoutSeconds 1, periodSeconds 10 and failureThreshold 3, and an HTTP readinessProbe on path /ready with initialDelaySeconds 0, periodSeconds 2, failureThreshold 3 and successThreshold 1; both probes use the named port http. Set resource requests to cpu 100m and memory 128Mi, and limits to cpu 1000m and memory 256Mi. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | Pod with resource requests (used by the scheduler) and limits (CPU is throttled, memory overuse is OOMKilled). requests < limits gives the Burstable QoS class. | [app-resources.yaml](yaml/app-resources.yaml) |
| app-secret-env | You are a Kubernetes expert. Generate a Kubernetes manifest (apiVersion v1) for a Pod named app-secret-env with restartPolicy Never and one container named mycontainer from image redis. The container gets two environment variables from an existing Secret named mysecret1 via secretKeyRef: SECRET_USERNAME from key username and SECRET_PASSWORD from key password. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | Pod that reads environment variables from a Secret via secretKeyRef. Depends on the Secret mysecret1 from [app-secret.yaml](yaml/app-secret.yaml): apply it first, otherwise the Pod gets stuck in CreateContainerConfigError. | [app-secret-env.yaml](yaml/app-secret-env.yaml) |
| app-secret | You are a Kubernetes expert. Generate a Kubernetes Secret manifest (apiVersion v1, type Opaque) named mysecret1 using stringData with keys username set to demo-user and password set to demo-password. Return only the manifest in a single yaml code block, without any explanations. Do not apply it to the cluster. | Supporting Secret for app-secret-env. stringData accepts plain text (Kubernetes base64-encodes it on save). Values are demo-only: real secrets should not be committed to git (use Sealed Secrets, External Secrets or SOPS). | [app-secret.yaml](yaml/app-secret.yaml) |

## Prompt engineering techniques used

Based on the [Prompt Engineering whitepaper by Google](https://www.kaggle.com/whitepaper-prompt-engineering):

- **Role prompting**: every prompt starts with "You are a Kubernetes expert".
- **Be specific**: resource kind, apiVersion, names, images, ports and probe parameters are stated explicitly, so the output is predictable.
- **Instructions over constraints**: the prompt says what to produce ("Return only the manifest in a single yaml code block").
- **Specify the output format**: a single YAML code block. kubectl-ai renders responses as Markdown, and plain-text YAML loses its indentation, while a code block keeps it intact.
- **Iterate and document**: the first attempt asked for "no markdown code fences" and produced broken YAML; the format instruction was changed based on that result. This README is the prompt log.
- **Fix known issues through the prompt**: deprecated APIs (batch/v1beta1), probe/port mismatches and missing dependencies (the Secret, generated by its own prompt) are addressed explicitly in the prompts.
- **One manifest per file**: each prompt produces exactly one resource, which keeps files simple to validate and review.

## How to reproduce

```zsh
export GEMINI_API_KEY=<your key>

gen() {
  local models=(gemini-3.6-flash gemini-3.5-flash gemini-3-flash-preview gemini-3.5-flash-lite)
  local m out
  for m in $models; do
    out=$(KUBECTL_AI_TERM_WIDTH=500 kubectl-ai --model $m --quiet "$1" 2>/dev/null)
    if [[ $out == *apiVersion* ]]; then
      print -r -- "$out" | sed -n '/apiVersion/,$p' | sed -E 's/^    //; s/ +$//' > "yaml/$2"
      echo "== model: $m"
      cat "yaml/$2"
      return 0
    fi
    echo "== $m failed, trying next..."
  done
  return 1
}

gen "<PROMPT from the table>" app.yaml
kubectl apply --dry-run=client -f yaml/
```

The `gen` helper falls back to the next model on errors (e.g. 503 under high demand), strips the kubectl-ai terminal formatting and saves the manifest only if YAML was actually returned. `KUBECTL_AI_TERM_WIDTH=500` disables Markdown word-wrapping, which would otherwise break long YAML lines.
