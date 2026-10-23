# Certificate installation for local services (Docker containers) on k3s

## Prerequisites

- Create a local `.env` file containing your environment-specific configuration.

Example `.env`:

```dotenv
DOMAIN=your-domain.org
```

Load the environment variables before running deployment commands:

```bash
set -a
source ../.env
set +a
```

## 1. Create a wildcard certificate

```bash
envsubst '${DOMAIN}' \
  < wildcard-certificate.yaml \
  | kubectl apply -f -
```
