# k8s-deploy-action

Eine itHub Action, die Kubernetes-Manifeste mit Kustomize erstellt und sie in einem Kubernetes-Cluster bereitstellt.

## Description

Diese action vereinfacht den Deployment Prozesses in Kubernetes, indem sie mehrere Schritte kombiniert und wiederverwendet werden kann:

1. Rendert Kubernetes manifests mit Kustomize
2. Fetches container image metadata to ensure deployment of the specific image digest
3. Deployt das gerenderte manifest in den angegebenen Kubernetes namespace


## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `folder` | Order in dem die Kustomize Konfiguration sich befindet | Yes | - |
| `image` | Container image das deployt werden soll | Yes | - |
| `namespace` | Kubernetes namespace in den deployt werden soll | Yes | - |
| `dry_run` | Kann gesetzt werden um ein wirkliches deployment zu verhindern (testen) | No | `""` (empty) |

## Verwendung

Folgende Konfiguration kann in den GitHub workflow eingebaut werden.
Hier eine Beispielkonfiguration aus dem uat deployment.

```yaml
jobs:
  deploy:
    steps:
      - uses: actions/checkout@v3
      
      # Set up Kubernetes configuration
      - name: Setup kubectl
        id: install-kubectl
        uses: azure/setup-kubectl@v4

      - uses: azure/k8s-set-context@v4
        with:
          method: kubeconfig
          kubeconfig: "${{ secrets.KUBECONFIG_UAT_PS }}"
      
      # Deploy to Kubernetes
      - name: deploy keycloak
        uses: intensivregister/k8s-deploy-action@v1.8
        with:
          namespace: ${{ env.NAMESPACE_KEYCLOAK }}
          folder: keycloak
          image: node-64e748724697219433d746b4.ps-xaas.io/intensivregister/keycloak-theme:${{ env.KEYCLOAK_THEME_VERSION }}
```

## Dry Run Option

Im Falle eines dry run muss die folgende Information ergänzt werden. Es wird kein Deployment durchgeführt.

```yaml
    dry_run: "true"
```

## Funktionsweise

1. Die action verwebdet [azure/k8s-bake](https://github.com/Azure/k8s-bake) zum Rendern des Kubernetes manifests unter Verwendung von Kustomize
2. `skopeo` wird verwendet um das Spezifizierte container image zu extrahieren und das entsprechende Digest zu erzeugen.
3. Das Manifest wird mithilfe von [Azure/k8s-deploy](https://github.com/Azure/k8s-deploy) auf dem Kubernetes Cluster deployed. Die version wird hierbei auf den spezifischen digest (...@sha11345..) gepinnt und ist somit immutable.
