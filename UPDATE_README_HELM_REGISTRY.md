# Documentation Update Instructions: Helm Registry Transition

## Context
The project has transitioned to the **Hyperledger Bevel** organization. As part of this move, the Helm chart release process has been updated. Newer versions (v1.14.0+) are now pushed to a new registry hosted on the `gh-pages` branch of the main repository.

## Current State of Registries
1.  **New Registry (Official)**: `https://hyperledger-bevel.github.io/bevel-operator-fabric/`
    - **Latest Version**: 1.14.0
    - **Other Versions**: 1.11.0
2.  **Old Registry (Legacy)**: `https://kfsoftware.github.io/hlf-helm-charts/`
    - **Latest Version**: 1.13.0 (Does not contain 1.14.0)

## Required Updates to `README.md`
Please update the "Install Kubernetes operator" section (and any other relevant sections) to point to the new registry.

### Recommended Change
Replace the old `helm repo add` and `helm install` commands:

**Old:**
```bash
helm repo add kfs https://kfsoftware.github.io/hlf-helm-charts --force-update
helm install hlf-operator --version=1.13.0 -- kfs/hlf-operator
```

**New:**
```bash
# Add the official Hyperledger Bevel registry
helm repo add hlf-operator https://hyperledger-bevel.github.io/bevel-operator-fabric/ --force-update

# Install the latest version (1.14.0)
helm install hlf-operator --version=1.14.0 hlf-operator/hlf-operator
```

## Technical Rationale
- Recent merges (`0b769963` and `75b970be`) updated the GitHub Actions to use `helm/chart-releaser-action` pointing to the `hyperledger-bevel` organization.
- The `gh-pages` branch on `upstream` confirms that `1.14.0` is only available in the new registry.
