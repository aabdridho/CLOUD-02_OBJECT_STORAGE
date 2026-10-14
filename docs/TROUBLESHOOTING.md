# CLOUD-02 — Troubleshooting Documentation

## 1. Overview

This document records the technical issues encountered during the implementation of CLOUD-02: Object Storage & Backup Recovery Lab.

The project was developed using Ubuntu WSL2, Docker Compose, SeaweedFS, and AWS CLI.

The purpose of this documentation is to describe the problems encountered, investigate their possible causes, record the solutions applied, and summarize the lessons learned.

---

## 2. Issue 01 — MinIO Docker Image Pull Failure

### Problem

The initial object storage implementation used MinIO. However, Docker failed to pull the container image during deployment.

### Error Message

```text
failed to resolve reference
"quay.io/minio/minio:RELEASE.2025-09-07T16-13-09Z"

401 UNAUTHORIZED
```

### Investigation

The Docker Compose configuration was checked using:

```bash
docker compose config --quiet
```

The configuration passed validation, but deployment failed when running:

```bash
docker compose up -d
```

This indicated that the failure occurred during image retrieval rather than YAML configuration validation.

### Root Cause Analysis

The container registry returned an HTTP 401 Unauthorized response when Docker attempted to retrieve the specified image.

The exact registry-side reason for denying access was not independently established during the investigation.

### Resolution

The local object storage implementation was changed from MinIO to SeaweedFS, which also provides an S3-compatible API.

The Docker Compose configuration was updated to use:

```yaml
image: chrislusf/seaweedfs:latest
```

### Verification

The replacement image was successfully downloaded, and the SeaweedFS container started.

Verification commands:

```bash
docker compose pull
docker compose up -d
docker compose ps
```

### Lesson Learned

Successful Docker Compose configuration validation does not guarantee successful deployment.

Container deployment also depends on image availability, registry accessibility, and runtime configuration.

---

## 3. Issue 02 — Duplicate YAML Mapping Key

### Problem

Docker Compose failed to validate the updated configuration after replacing MinIO with SeaweedFS.

### Error Message

```text
yaml: unmarshal errors:
line 39: mapping key "volumes" already defined at line 19
```

### Root Cause Analysis

The configuration contained duplicate `volumes` keys at the same YAML mapping level.

This was encountered during the transition from the previous MinIO configuration to the new SeaweedFS configuration.

### Resolution

The Docker Compose file was reorganized to separate service-level volume mounting from top-level volume definitions.

Correct structure:

```yaml
services:
  seaweedfs:
    volumes:
      - seaweedfs_data:/data

volumes:
  seaweedfs_data:
```

### Verification

```bash
docker compose config --quiet
```

### Lesson Learned

YAML is indentation-sensitive, and mapping keys must be defined within the correct hierarchy.

A service-level `volumes` declaration and a top-level `volumes` declaration serve different purposes.

---

## 4. Issue 03 — Invalid Docker Compose Volume Structure

### Problem

After correcting the duplicate mapping issue, Docker Compose reported another configuration validation error.

### Error Message

```text
volumes additional properties
'seaweedfs_data:services' not allowed
```

### Root Cause Analysis

Docker Compose interpreted `seaweedfs_data:services` as an invalid property within the top-level volume configuration.

This suggested an incorrectly structured YAML mapping, potentially caused by merged text or incorrect formatting during editing.

### Resolution

The `compose.yaml` configuration was rewritten with the correct YAML hierarchy.

The corrected configuration contained a single top-level `services` section and a separate top-level `volumes` section.

### Verification

The configuration passed validation:

```bash
docker compose config --quiet
```

SeaweedFS was subsequently deployed successfully.

### Lesson Learned

Infrastructure configuration errors can occur even when the individual configuration values are correct.

Correct indentation, mapping structure, and syntax validation are essential before deployment.

---

## 5. Issue 04 — S3 API Returned HTTP 403

### Observation

After deploying SeaweedFS, the S3 endpoint was tested using:

```bash
curl -i http://127.0.0.1:8333/
```

### Response

```text
HTTP/1.1 403 Forbidden
```

The response included an S3-style `AccessDenied` message.

### Analysis

The request was sent without S3 authentication credentials.

The HTTP 403 response demonstrated that the endpoint was reachable but did not authorize the anonymous request.

This was an expected authentication behavior rather than evidence of a failed deployment.

### Verification

AWS CLI was configured with the local S3 credentials and endpoint.

```bash
aws --endpoint-url http://127.0.0.1:8333 s3 ls
```

The authenticated request successfully listed the two configured buckets:

- `primary-data`
- `backup-data`

Object upload and bucket operations were subsequently tested successfully.

### Lesson Learned

An HTTP 403 response does not necessarily indicate that a service is unavailable.

Troubleshooting should distinguish between network connectivity, application availability, authentication, and authorization.

---

## 6. Issue 05 — Git Repository Not Initialized

### Problem

During repository preparation, Git failed to evaluate the `.gitignore` configuration.

### Error Message

```text
fatal: not a git repository
```

### Root Cause

The project directory had not yet been initialized as a Git repository.

### Resolution

The following commands were used:

```bash
git init
git branch -M main
```

### Verification

```bash
git status
git check-ignore .env
```

The repository was successfully initialized, and `.env` was recognized as an ignored file.

### Lesson Learned

Git commands that depend on repository metadata require an initialized repository.

---

## 7. Issue 06 — Temporary Files Included in Git Staging

### Problem

Microsoft Office temporary files were unintentionally included when staging project files with `git add .`.

### Investigation

The staged files were reviewed using:

```bash
git status
```

Temporary files with `.tmp` and Microsoft Office lock-file naming patterns were identified.

### Resolution

Temporary files were removed from the Git index without deleting the local files.

Example:

```bash
git rm --cached -- '~WRL3254.tmp'
```

The `.gitignore` file was configured with:

```gitignore
.env
.env.*
!.env.example

restored-data/

~$*
*.tmp

.DS_Store
Thumbs.db
```

### Verification

```bash
git check-ignore .env
git status --short
```

The final staged file list no longer included `.env` or temporary files.

### Lesson Learned

Before committing a project, developers should review all staged files to avoid including credentials, generated artifacts, or unnecessary temporary files.

---

## 8. Troubleshooting Summary

| Issue | Category | Outcome |
|---|---|---|
| MinIO image pull — 401 Unauthorized | Container Registry | Changed to SeaweedFS |
| Duplicate YAML mapping | Configuration | Resolved |
| Invalid volume structure | Configuration | Resolved |
| S3 API 403 AccessDenied | Authentication | Expected behavior; authenticated access worked |
| Git repository not initialized | Version Control | Resolved |
| Temporary files staged | Repository Hygiene | Resolved |

## 9. Key Learning Outcomes

Through this project, I gained practical experience in:

- Diagnosing container image retrieval failures.
- Understanding Docker Compose configuration validation.
- Identifying YAML structure and indentation errors.
- Differentiating HTTP authentication errors from connectivity failures.
- Testing S3-compatible storage endpoints.
- Preparing Git repositories safely.
- Protecting environment credentials from accidental commits.

## 10. Conclusion

The CLOUD-02 troubleshooting process demonstrated the importance of validating each infrastructure component independently.

Instead of assuming that every deployment failure originates from the application, the investigation considered container registry access, configuration structure, service availability, authentication, and version control.

These experiences helped strengthen foundational Cloud Engineering troubleshooting skills.
