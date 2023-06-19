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
 --interval=1m \
 --export > ./deploy/flux_sync.yaml