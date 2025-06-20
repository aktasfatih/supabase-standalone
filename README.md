# Supabase Kubernetes Helm Chart

A comprehensive Helm chart for deploying [Supabase](https://supabase.com) on Kubernetes.

## Overview

This Helm chart deploys a complete Supabase stack on Kubernetes, including:

- **PostgreSQL Database** - Built-in database with Supabase extensions
- **Auth Service** - User authentication and authorization
- **REST API** - Auto-generated REST APIs for your database
- **Realtime** - Websocket server for real-time subscriptions
- **Storage** - S3-compatible object storage
- **Kong Gateway** - API gateway for routing and plugins
- **Studio** - Web-based database management UI
- **Additional Services** - Analytics, Functions, Image Proxy, and more

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure
- (Optional) NGINX Ingress Controller for external access
- (Optional) cert-manager for TLS certificates

## Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/supabase-kubernetes.git
cd supabase-kubernetes
```

### 2. Generate JWT keys

Before deploying, you need to generate JWT keys for Supabase:

```bash
# Using Supabase CLI
supabase bootstrap gen

# Or use the online tool
# Visit: https://supabase.com/docs/guides/self-hosting/docker#generate-api-keys
```

### 3. Create your values file

```bash
cp values.yaml my-values.yaml
```

Edit `my-values.yaml` and update these critical values:

```yaml
supabase:
  secret:
    jwt:
      anonKey: "your-generated-anon-key"
      serviceKey: "your-generated-service-key"
      secret: "your-jwt-secret-with-at-least-32-characters"
    db:
      password: "change-this-super-secret-password"
    analytics:
      apiKey: "your-analytics-api-key"
  
  # Update these URLs
  studio:
    environment:
      SUPABASE_PUBLIC_URL: "https://api.yourdomain.com"
  auth:
    environment:
      API_EXTERNAL_URL: "https://api.yourdomain.com"
      GOTRUE_SITE_URL: "https://yourdomain.com"
      GOTRUE_SMTP_HOST: "smtp.yourdomain.com"
      GOTRUE_SMTP_ADMIN_EMAIL: "admin@yourdomain.com"
      GOTRUE_SMTP_SENDER_NAME: "noreply@yourdomain.com"
  kong:
    ingress:
      hosts:
        - host: api.yourdomain.com
          paths:
            - path: /
              pathType: Prefix
      tls:
        - secretName: supabase-tls
          hosts:
            - api.yourdomain.com
```

### 4. Install the chart

```bash
# Update dependencies
helm dependency update

# Install
helm install supabase . -n supabase --create-namespace -f my-values.yaml

# Or upgrade existing installation
helm upgrade supabase . -n supabase -f my-values.yaml
```

### 5. Access Supabase

Once deployed, you can access Supabase:

```bash
# Get the Kong (API) URL
kubectl get ingress -n supabase

# Port-forward to Studio (if ingress not configured)
kubectl port-forward -n supabase svc/supabase-supabase-studio 3000:3000
# Visit http://localhost:3000
```

## Configuration

### Essential Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `supabase.secret.jwt.anonKey` | Anonymous JWT key | `YOUR-ANON-KEY` |
| `supabase.secret.jwt.serviceKey` | Service role JWT key | `YOUR-SERVICE-KEY` |
| `supabase.secret.jwt.secret` | JWT secret (min 32 chars) | `YOUR-SUPER-SECRET...` |
| `supabase.secret.db.password` | Database password | `your-super-secret-password` |
| `supabase.kong.ingress.hosts[0].host` | Your domain | `supabase.yourdomain.com` |

### Database Configuration

```yaml
supabase:
  db:
    enabled: true
    persistence:
      enabled: true
      size: 8Gi
      storageClass: ""  # Uses default storage class
```

### SMTP Configuration (for Auth emails)

```yaml
supabase:
  secret:
    smtp:
      username: "your-smtp-username"
      password: "your-smtp-password"
  auth:
    environment:
      GOTRUE_SMTP_HOST: "smtp.sendgrid.net"
      GOTRUE_SMTP_PORT: "587"
      GOTRUE_SMTP_ADMIN_EMAIL: "admin@yourdomain.com"
      GOTRUE_SMTP_SENDER_NAME: "noreply@yourdomain.com"
```

### Storage Configuration

```yaml
supabase:
  storage:
    persistence:
      enabled: true
      size: 10Gi
    environment:
      FILE_SIZE_LIMIT: "52428800"  # 50MB in bytes
      # For S3 backend:
      # STORAGE_BACKEND: "s3"
      # GLOBAL_S3_BUCKET: "your-bucket"
      # AWS_DEFAULT_REGION: "us-east-1"
```

### Backup Configuration

```yaml
supabase:
  backup:
    enabled: true
    schedule: "0 2 * * *"  # Daily at 2 AM
    retentionDays: 30
    storage:
      size: 50Gi
    # Optional S3 backups
    s3:
      enabled: true
      bucket: "your-backup-bucket"
      accessKeyId: "your-access-key"
      secretAccessKey: "your-secret-key"
```

### TLS/SSL Configuration

For production, enable TLS:

```yaml
supabase:
  kong:
    ingress:
      annotations:
        cert-manager.io/cluster-issuer: "letsencrypt-prod"
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: supabase-tls
          hosts:
            - api.yourdomain.com
```

## Production Deployment

### Security Checklist

- [ ] Generate new JWT keys (don't use defaults)
- [ ] Set strong database password
- [ ] Configure SMTP for email verification
- [ ] Enable TLS/SSL certificates
- [ ] Set resource limits for pods
- [ ] Configure backup retention
- [ ] Review and adjust storage sizes
- [ ] Enable network policies if required

### High Availability

For HA deployments, consider:

1. **Database**: Use an external managed database (RDS, Cloud SQL, etc.)
2. **Storage**: Use S3-compatible storage instead of local files
3. **Replicas**: Increase replica counts for stateless services

Example HA configuration:

```yaml
supabase:
  # Disable built-in DB
  db:
    enabled: false
  
  # Point to external database
  auth:
    environment:
      DB_HOST: "your-external-db.amazonaws.com"
  
  # Increase replicas
  auth:
    replicaCount: 3
  rest:
    replicaCount: 3
  realtime:
    replicaCount: 2
```

### Monitoring

The chart includes Vector for log aggregation. For full monitoring:

1. Deploy Prometheus and Grafana
2. Configure Vector to ship logs to your logging system
3. Set up alerts for critical services

## Troubleshooting

### Common Issues

1. **Pods not starting**: Check PVC status
   ```bash
   kubectl get pvc -n supabase
   kubectl describe pod <pod-name> -n supabase
   ```

2. **Auth emails not sending**: Verify SMTP configuration
   ```bash
   kubectl logs -n supabase deployment/supabase-supabase-auth
   ```

3. **Can't access Studio**: Check ingress and services
   ```bash
   kubectl get ingress,svc -n supabase
   ```

### Debug Commands

```bash
# Check all pods
kubectl get pods -n supabase

# Check logs
kubectl logs -n supabase -l app.kubernetes.io/name=supabase

# Check events
kubectl get events -n supabase --sort-by='.lastTimestamp'

# Describe a failing pod
kubectl describe pod <pod-name> -n supabase
```

## Backup and Restore

### Manual Backup

```bash
# Create a manual backup
kubectl create job --from=cronjob/supabase-supabase-backup manual-backup -n supabase

# Check backup status
kubectl logs -n supabase job/manual-backup
```

### Restore from Backup

```bash
# Copy backup from PVC
kubectl cp -n supabase <backup-pod>:/backups/supabase_backup_20240120_020000.dump ./backup.dump

# Restore to database
kubectl exec -it -n supabase supabase-supabase-db-0 -- pg_restore -U postgres -d postgres /path/to/backup.dump
```

## Upgrading

### Chart Upgrade

```bash
# Update repo
helm repo update

# Check changes
helm diff upgrade supabase . -n supabase -f my-values.yaml

# Perform upgrade
helm upgrade supabase . -n supabase -f my-values.yaml
```

### Supabase Version Upgrade

Update image tags in your values file:

```yaml
supabase:
  auth:
    image:
      tag: "v2.171.0"  # Check Supabase releases
  rest:
    image:
      tag: "v12.2.11"
  # ... update other services
```

## Uninstall

```bash
# Uninstall chart (keeps PVCs)
helm uninstall supabase -n supabase

# Delete namespace and all resources
kubectl delete namespace supabase

# Note: PVCs may need manual deletion
kubectl delete pvc -n supabase --all
```

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## Support

- [Supabase Documentation](https://supabase.com/docs)
- [Supabase GitHub](https://github.com/supabase/supabase)
- [Chart Issues](https://github.com/yourusername/supabase-kubernetes/issues)

## License

This chart is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details.

Supabase itself is licensed under the [Apache 2.0 License](https://github.com/supabase/supabase/blob/master/LICENSE).