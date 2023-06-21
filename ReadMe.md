# Account used by fluxcd to communicate with SOT.
export GITHUB_USER=biraderomkar

export GITHUB_TOKEN=""

# Bootstrapping fluxcd toolkits in k8s cluster

flux bootstrap github \
 --owner=$GITHUB_USER \
 --repository=fluxcdInfraSOT \
 --branch=main \
 --path=deployinfra \
 --personal

# Define source SOT
flux create source git nginxappsot \
 --url=https://github.com/biraderomkar/nginxAppSOT.git \
 --branch=main \
 --interval=30s \
 --export > ./deploy/flux_source.yaml

# Apply above source SOT using kustomization controller
 flux create kustomization nginxappsot \
 --source=nginxappsot  \
 --path="./deploy/" \
 --prune=true \
 --validation=client \
 --interval=30s  \
 --export > ./deploy/flux_sync.yaml

 # Setting Up prometheus stack
 flux create source git flux-monitoring \
  --interval=30m \
  --url=https://github.com/fluxcd/flux2 \
  --branch=main \
  --export > ./deploy/flux_monitoring_source.yaml

# Apply kube-prom-stack using below kustomization definition
flux create kustomization kube-prometheus-stack \
  --interval=1h \
  --prune \
  --source=flux-monitoring \
  --path="./manifests/monitoring/kube-prometheus-stack" \
  --health-check-timeout=5m \
  --wait \
  --export > ./deploy/flux_monitoring_sync.yaml