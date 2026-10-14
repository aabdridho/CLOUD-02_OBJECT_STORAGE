# CLOUD-02 — Object Storage & Backup Recovery Lab

## Overview

A beginner-level Cloud Engineering project focused on understanding object storage, S3-compatible APIs, backup procedures, data restoration, and integrity verification.

The project uses SeaweedFS as a local object storage service, Docker Compose for container management, and AWS CLI for object operations.

## Objectives

- Understand object storage architecture.
- Deploy an S3-compatible storage service.
- Perform authenticated bucket and object operations.
- Simulate backup and accidental object deletion.
- Restore objects from a backup bucket.
- Verify file integrity using SHA-256.
- Test data persistence across container recreation.

## Technology Stack

| Technology | Purpose |
|---|---|
| Ubuntu WSL2 | Linux development environment |
| Docker | Container runtime |
| Docker Compose | Service configuration and deployment |
| SeaweedFS | S3-compatible object storage |
| AWS CLI | Storage operations through S3 API |
| SHA-256 | File integrity verification |

## Architecture

```text
Ubuntu WSL2
    |
    | AWS CLI / S3 API
    v
SeaweedFS (Docker)
    |
    +--- primary-data
    |       └── text.txt
    |
    +--- backup-data
            └── text.txt
```

Both buckets are hosted on the same SeaweedFS instance. This setup demonstrates logical backup and recovery, not full disaster recovery.

## Getting Started

### Prerequisites

- Linux or Ubuntu WSL2
- Docker Engine
- Docker Compose
- AWS CLI

### Configuration

1. Clone the repository.
2. Create a `.env` file based on `.env.example`.
3. Replace placeholder credentials with unique local credentials.
4. Start the storage service.

```bash
docker compose config --quiet
docker compose up -d
docker compose ps
```

### Configure AWS CLI

```bash
set -a
source .env
set +a

export AWS_ENDPOINT_URL=http://127.0.0.1:8333
export AWS_DEFAULT_REGION=us-east-1
```

### List Buckets

```bash
aws s3 ls
```

### Upload an Object

```bash
aws s3 cp sample-data/text.txt s3://primary-data/text.txt
```

### Create a Backup

```bash
aws s3 cp s3://primary-data/text.txt s3://backup-data/text.txt
```

### Restore an Object

```bash
aws s3 cp s3://backup-data/text.txt s3://primary-data/text.txt
```

### Verify Data Integrity

```bash
aws s3 cp s3://primary-data/text.txt restored-data/text.txt

sha256sum sample-data/text.txt
sha256sum restored-data/text.txt

cmp sample-data/text.txt restored-data/text.txt
```

Matching SHA-256 hashes and a successful byte-level comparison indicate that the restored file matches the original.

## Testing

| Test | Expected Result |
|---|---|
| Docker deployment | Container is running |
| S3 endpoint | HTTP endpoint responds |
| Authentication | Bucket listing succeeds |
| Object upload | Object appears in primary bucket |
| Backup | Object appears in backup bucket |
| Simulated deletion | Primary object is removed |
| Restore | Primary object is recovered |
| Integrity verification | Original and restored contents match |
| Persistence test | Objects remain after container recreation |

Actual test results and evidence should be recorded separately.

## Limitations

- Local deployment only.
- Primary and backup buckets share one storage instance.
- No off-site backup.
- No automated backup scheduling.
- No production-grade high availability.

## Learning Outcomes

This project provides hands-on experience with object storage fundamentals, Docker persistence, S3-compatible APIs, backup and restore procedures, and troubleshooting storage services.

## Documentation

- [Architecture](docs/ARCHITECTURE.md)
- [Troubleshooting](docs/TROUBLESHOOTING.md)
- [Interview Preparation](docs/INTERVIEW_QA.md)
