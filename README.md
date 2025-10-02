# MinIO no Rancher + StorageClass S3 via CSI (Argo CD)

Este repositório contém manifests do Argo CD para:
- **MinIO** (implantado via Helm Chart).
- **CSI S3** (exemplo com `ru.yandex.s3.csi` e `ctrox/csi-s3`).
- **StorageClass**, **PVC** e **Pod de teste**.

> ⚠️ **Importante**: MinIO é armazenamento de **objetos (S3)**. Para montar em Pods como PV/PVC
precisa de um **driver CSI S3** (FUSE). Para workloads sensíveis a POSIX/performance,
considere NFS/Ceph/Block.

## Estrutura

```
apps/
  app-of-apps.yaml           # Application "raiz" (padrão app-of-apps)
  minio/
    application.yaml         # ArgoCD Application apontando para o Helm chart do MinIO
    helm-values.example.yaml # Valores de exemplo (não referenciados pela app)
  csi-s3/
    application.yaml         # ArgoCD Application para o CSI e configs
    kustomization.yaml
    secret.example.yaml      # credenciais MinIO para o CSI
    configmap.example.yaml   # endpoint/region/pathStyle
    storageclass.yaml        # StorageClass minio-s3-csi
  examples/
    application.yaml         # ArgoCD Application para exemplos
    pvc.yaml                 # PVC usando o StorageClass minio-s3-csi
    pod.yaml                 # Pod que monta o PVC
```

## Uso rápido (GitOps)

1. **Crie um repositório** no GitHub (ex.: `sdtitecnolog/minio-rancher-argocd`).
2. **Envie estes arquivos** para o repositório.
3. No **Argo CD** (no cluster gerenciado pelo Rancher):
   - Adicione o Git repo em *Settings → Repositories* (URL + auth).
   - Crie uma aplicação apontando para `apps/app-of-apps.yaml` **ou** aplique via `kubectl`:
     ```bash
     kubectl apply -f apps/app-of-apps.yaml -n argocd
     ```
4. Ajuste os arquivos `secret.example.yaml` e `configmap.example.yaml` (substitua `<PLACEHOLDERS>`), renomeie para `secret.yaml` e `configmap.yaml` e **commite**.
5. Aguarde o Argo CD sincronizar. Verifique:
   - Serviço do MinIO (LoadBalancer/Ingress).
   - CSI S3 Controller/Nodes ativos.
   - PVC e Pod de exemplo sincronizados.

### Endpoint do MinIO
- Caso use LoadBalancer: `http://<EXTERNAL-IP>:9000`
- Caso use Ingress (configure no Helm do MinIO): `https://minio.<SEU-DOMINIO>`

> **Dica**: Crie o bucket (ex.: `k8s-volumes`) no MinIO via `mc` ou UI antes do PVC ser criado.
