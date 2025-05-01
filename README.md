# PNPM deploy inconsistent output

This repository demonstrates a reproducible issue in [`pnpm deploy`](https://pnpm.io/cli/deploy) that causes cache invalidation due to non-deterministic timestamps in the generated `.modules.yaml` file.

## Problem Description

When running `pnpm deploy`, even if the source packages and lockfile remain unchanged, the resulting `node_modules` tree includes a `.modules.yaml` file with a different `prunedAt` timestamp every time the command is executed. This breaks Docker caching layers and results in unnecessary image rebuilds, which in turn leads to longer build times and increased resource usage.

This is problematic in production environments where deterministic builds are expected and caching is critical.

## Steps to Reproduce

1. Clone this repository or use a codespace.

```bash
git clone https://github.com/kerolloz/pnpm-deploy-issue
cd pnpm-deploy-issue
```

2. Install and Run the deploy command twice:

```bash
pnpm install
pnpm deploy -F p1 deployment-build/p1
mv deployment-build/p1 deployment-build/old-build # Keep a copy of the first build
sleep 3 # Wait a few seconds to ensure a different timestamp
pnpm deploy -F p1 deployment-build/p1
```

3. Compare the two outputs:

```bash
diff -qr deployment-build/p1 deployment-build/old-build
```

4. Observe that the only difference is in the `.modules.yaml` file. So let's check the diff of the two files:

```bash
diff deployment-build/p1/node_modules/.modules.yaml deployment-build/old-build/node_modules/.modules.yaml
```

## Example run of the reproduction steps

![image](https://github.com/user-attachments/assets/a11a5b4d-425f-4c2c-960e-149364a3f5a3)
