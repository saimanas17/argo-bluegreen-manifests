# Kubernetes Manifests for Blue-Green Deployment

GitOps repository with Kubernetes manifests for blue-green deployment using Argo Rollouts.

## 📂 Structure
```
argo-bluegreen-manifests/
├── rollout.yaml                # Argo Rollouts config
├── service-active.yaml         # Production (port 30080)
├── service-preview.yaml        # Preview (port 30081)
├── argocd-application.yaml     # ArgoCD Application
└── README.md
```

## 🔑 Key Configuration

**rollout.yaml**:
- `replicas: 2` - High availability
- `autoPromotionEnabled: false` - Manual approval required
- `scaleDownDelaySeconds: 30` - Wait before scaling down old version
- `activeService` - Production traffic (30080)
- `previewService` - Preview traffic (30081)

**Sync Policy**:
- Automated sync enabled
- Auto-prune deleted resources
- Self-heal if cluster drifts from Git

## 🔄 Deployment Flow
```
Jenkins updates image tag in rollout.yaml
              ↓
Commits and pushes to this repo
              ↓
GitHub webhook triggers ArgoCD
              ↓
ArgoCD syncs to Kubernetes
              ↓
Argo Rollouts creates preview
   - New pods (preview service)
   - Old pods (active service)
              ↓
Manual testing on preview
              ↓
Promotion → Active service switches to new pods
              ↓
Old pods scaled down after 30s
```

## 🚀 Setup
```bash
# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Install kubectl plugin
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x kubectl-argo-rollouts-linux-amd64
sudo mv kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts

# Apply manifests
kubectl apply -f rollout.yaml
kubectl apply -f service-active.yaml
kubectl apply -f service-preview.yaml
kubectl apply -f namespace.yaml
```

### GitHub Webhook

Settings → Webhooks → Add webhook:
```
Payload URL: https://<ARGOCD_SERVER>/api/webhook
Content type: application/json
Events: Just the push event
```

## 🎮 Management Commands
```bash
# View rollout status
kubectl argo rollouts get rollout bluegreen-frontend -n bluegreen-demo

# Watch live
kubectl argo rollouts get rollout bluegreen-frontend -n bluegreen-demo --watch

# Promote to production
kubectl argo rollouts promote bluegreen-frontend -n bluegreen-demo

# Rollback
kubectl argo rollouts undo bluegreen-frontend -n bluegreen-demo

# Rollback to specific revision
kubectl argo rollouts undo bluegreen-frontend -n bluegreen-demo --to-revision=2

# View history
kubectl argo rollouts history bluegreen-frontend -n bluegreen-demo

# Abort rollout
kubectl argo rollouts abort bluegreen-frontend -n bluegreen-demo

# Restart
kubectl argo rollouts restart bluegreen-frontend -n bluegreen-demo
```

## 📊 Monitoring
```bash


# Get password:
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Check services
kubectl get svc -n bluegreen-demo

# Check pods
kubectl get pods -n bluegreen-demo

# Pod logs
kubectl logs -n bluegreen-demo -l app=bluegreen-frontend --tail=50
```

## 🔗 Related

- [Application Repository](https://github.com/saimanas17/argocd-bluegreen-app)
- [Parent Repository](https://github.com/saimanas17/argocd-bluegreen)
- [Argo Rollouts Docs](https://argoproj.github.io/argo-rollouts/)
- [ArgoCD Docs](https://argo-cd.readthedocs.io/)