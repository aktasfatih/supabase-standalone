# Contributing to Supabase Kubernetes Helm Chart

First off, thank you for considering contributing to this project! 🎉

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When creating a bug report, please include:

- **Chart version** you're using
- **Kubernetes version** and environment (EKS, GKE, AKS, bare-metal, etc.)
- **Values file** (sanitized of sensitive data)
- **Error messages** and logs
- **Steps to reproduce** the issue

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. When creating an enhancement suggestion, please include:

- **Use case** - why is this enhancement needed?
- **Proposed solution** - how should it work?
- **Alternative solutions** you've considered

### Pull Requests

1. Fork the repo and create your branch from `main`
2. Make your changes
3. Test your changes thoroughly
4. Update documentation if needed
5. Submit a pull request

## Development Setup

### Prerequisites

- Kubernetes cluster (minikube, kind, or cloud provider)
- Helm 3.x
- kubectl
- Optional: helm-docs for generating documentation

### Testing Your Changes

1. **Lint the chart:**
   ```bash
   helm lint .
   ```

2. **Template rendering test:**
   ```bash
   helm template test . -f values.yaml
   ```

3. **Dry run installation:**
   ```bash
   helm install test . --dry-run --debug
   ```

4. **Full installation test:**
   ```bash
   # Create test namespace
   kubectl create namespace test-supabase
   
   # Install
   helm install test . -n test-supabase
   
   # Check pods
   kubectl get pods -n test-supabase
   
   # Clean up
   helm uninstall test -n test-supabase
   kubectl delete namespace test-supabase
   ```

## Code Standards

### Helm Best Practices

- Follow [Helm best practices](https://helm.sh/docs/chart_best_practices/)
- Use `helm lint` to check for issues
- Keep templates readable and well-commented
- Use `_helpers.tpl` for repeated template snippets

### Values Structure

- Document all values with comments
- Provide sensible defaults where possible
- Group related values logically
- Use consistent naming conventions

### Documentation

- Update README.md for user-facing changes
- Add inline comments for complex template logic
- Update values.yaml comments for new parameters
- Include examples for new features

## Commit Guidelines

We follow conventional commits:

- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `chore:` - Maintenance tasks
- `test:` - Test additions/changes

Examples:
```
feat: add S3 storage backend support
fix: correct database connection string for external DB
docs: update backup configuration examples
chore: bump postgres image to 15.8.1.061
```

## Testing Checklist

Before submitting a PR, ensure:

- [ ] Chart installs successfully with default values
- [ ] Chart upgrades work from previous version
- [ ] All pods become ready
- [ ] Basic functionality works (can access Studio, create tables, etc.)
- [ ] Uninstall removes all resources cleanly
- [ ] Documentation is updated
- [ ] `helm lint` passes
- [ ] Sensitive values are not exposed in templates

## Release Process

1. Update version in `Chart.yaml`
2. Update `artifacthub.io/changes` annotation
3. Update README.md if needed
4. Create a git tag matching the version
5. Package the chart: `helm package .`

## Getting Help

- Check existing [issues](https://github.com/yourusername/supabase-kubernetes/issues)
- Review [Supabase documentation](https://supabase.com/docs)
- Ask in [Supabase Discord](https://discord.supabase.com/)

## Code of Conduct

Please note we have a code of conduct - be respectful and constructive in all interactions.

Thank you for contributing! 🚀