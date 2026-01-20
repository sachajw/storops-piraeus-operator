# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Piraeus Operator v2 is a Kubernetes Operator written in Go that manages LINSTOR storage clusters. It automates deployment of DRBD, LINSTOR, LINSTOR CSI driver, and LINSTOR High-Availability Controller.

## Build Commands

```bash
# Development
make manifests      # Generate CRDs, RBAC, webhooks
make generate       # Generate DeepCopy methods
make fmt            # Run go fmt
make vet            # Run go vet
make build          # Build bin/manager

# Testing
make test           # Run full test suite with coverage
make compat-test    # Run tests against Kubernetes 1.20

# Docker
make docker-build   # Build container image
make docker-push    # Build and push to quay.io/piraeusdatastore/piraeus-operator

# Deployment (to current kubectl context)
make install        # Install CRDs
make deploy         # Deploy operator
make uninstall      # Remove CRDs
make undeploy       # Remove operator
```

### Running a Single Test

```bash
# Run specific test file
go test ./internal/controller/... -run TestLinstorCluster

# Run with ginkgo focus
go test ./internal/controller/... -ginkgo.focus="should create"

# Run specific package tests
go test ./pkg/linstorhelper/...
```

## Architecture

### Custom Resources (api/v1/)

- **LinstorCluster**: Top-level resource defining a LINSTOR cluster configuration
- **LinstorSatellite**: Per-node storage satellite configuration
- **LinstorSatelliteConfiguration**: Shared satellite settings applied via label selectors
- **LinstorNodeConnection**: Inter-node connection policies

### Controllers (internal/controller/)

Controllers reconcile CRD state against actual cluster state:
- `LinstorClusterReconciler`: Manages controller/CSI deployments, creates LinstorSatellite resources
- `LinstorSatelliteReconciler`: Manages per-node DaemonSets, integrates with ClusterAPI
- `LinstorNodeConnectionReconciler`: Configures DRBD paths between nodes

### Key Packages

- **pkg/resources/**: Resource templates for cluster and satellite components. Uses embedded YAML manifests loaded at runtime.
- **pkg/linstorhelper/**: LINSTOR API client wrapper with rate limiting and caching
- **pkg/merge/**: Merges LinstorSatelliteConfiguration resources based on label selectors
- **pkg/utils/**: JSON patching, property resolution, version comparisons

### Webhooks (internal/webhook/)

Validation webhooks ensure CRD correctness before persistence. Storage class webhook validates CSI parameters.

## Testing

Uses Ginkgo v2 + Gomega with envtest for controller integration tests. Test suites bootstrap a real API server and etcd.

```go
// Common test pattern
var _ = Describe("LinstorCluster", func() {
    It("should create satellite resources", func() {
        // Use Eventually/Consistently for async assertions
        Eventually(func() bool {
            // check condition
        }).Should(BeTrue())
    })
})
```

## Code Style

- Uses gofumpt (stricter gofmt)
- golangci-lint with staticcheck
- Generated files (`zz_generated*.go`) are excluded from linting
- Commits require DCO sign-off (`Signed-off-by:` line)

## Key Dependencies

- `github.com/LINBIT/golinstor`: LINSTOR client library
- `sigs.k8s.io/controller-runtime`: Kubernetes operator framework
- `github.com/cert-manager/cert-manager`: TLS certificate management
- `sigs.k8s.io/cluster-api`: Optional ClusterAPI machine integration
