# CLOUD-02 — System Architecture

## 1. Architecture Overview

CLOUD-02 is a local object storage and backup recovery laboratory designed to demonstrate fundamental Cloud Engineering concepts.

The system uses Ubuntu WSL2 as the development environment, Docker Compose for container orchestration, SeaweedFS as the S3-compatible object storage server, and AWS CLI as the client for storage operations.

The architecture consists of a single storage instance with two logical buckets: `primary-data` and `backup-data`.

## 2. Architecture Diagram

```text
+--------------------------------------+
|            Ubuntu WSL2               |
|                                      |
|        AWS CLI (S3 Client)           |
+------------------+-------------------+
                   |
                   | HTTP / S3 API
                   | 127.0.0.1:8333
                   v
+--------------------------------------+
|       Docker Container               |
|       cloud02-seaweedfs              |
|                                      |
|        SeaweedFS S3 Service          |
|                                      |
|     +-------------+-------------+    |
|     |             |             |    |
|     v             v             |    |
|  primary-data   backup-data     |    |
|   text.txt       text.txt       |    |
|     |             |             |    |
+-----+-------------+-------------+----+
                   |
                   v
+--------------------------------------+
|        Docker Named Volume           |
|            seaweedfs_data            |
|        Mounted at /data              |
+--------------------------------------+
```

## 3. Technology Components

### Ubuntu WSL2

Ubuntu WSL2 provides a Linux environment running on Windows.

It is used to execute Linux commands, manage Docker containers, and interact with the storage system.

### Docker & Docker Compose

Docker provides containerized execution for SeaweedFS.

Docker Compose defines the container image, port mapping, environment variables, and persistent storage.

### SeaweedFS

SeaweedFS is an object storage system that supports S3-compatible API operations.

In this project, SeaweedFS manages objects stored in the `primary-data` and `backup-data` buckets.

### AWS CLI

AWS CLI is used as an S3-compatible client for uploading, downloading, listing, copying, and deleting objects.

Requests are directed to the local SeaweedFS endpoint instead of Amazon S3.

## 4. Network Architecture

The S3 API is exposed using:

`http://127.0.0.1:8333`

The Docker port mapping is:

`127.0.0.1:8333:8333`

Binding to the loopback interface restricts access through the published host port to local connections.

HTTP is used only for this local learning environment. A production deployment would require additional security controls, including HTTPS and appropriately restricted network access.

## 5. Storage Architecture

The storage system uses two logical buckets.

**Primary Bucket — `primary-data`**

Stores the active object used during normal operations.

**Backup Bucket — `backup-data`**

Stores a copy of the object for simulated recovery.

Both buckets belong to the same SeaweedFS instance and share the underlying storage infrastructure.

The Docker named volume `seaweedfs_data` is mounted at `/data` to preserve stored data across container recreation.

## 6. Backup Workflow

1. Upload `text.txt` to the primary bucket.
2. Confirm that the object is available.
3. Copy the object from the primary bucket to the backup bucket.
4. Verify that the backup object exists.

**Command:**

`aws s3 cp s3://primary-data/text.txt s3://backup-data/text.txt`

## 7. Recovery Workflow

1. Simulate accidental deletion of `text.txt` from the primary bucket.
2. Confirm that the backup object remains available.
3. Copy the object from the backup bucket to the primary bucket.
4. Download the restored object.
5. Compare the restored file with the original.

**Recovery Command:**

`aws s3 cp s3://backup-data/text.txt s3://primary-data/text.txt`

## 8. Data Integrity Verification

SHA-256 is used to verify the integrity of restored data.

The original file and restored file are hashed independently.

If the hashes match, this provides strong evidence that the file contents are identical.

A byte-level comparison using `cmp` provides an additional verification method.

## 9. Persistence Mechanism

The project uses a Docker named volume to separate persistent storage from the container lifecycle.

When the container is removed and recreated without deleting the volume, the stored data should remain available.

This behavior can be tested using:

- `docker compose down`
- `docker compose up -d`
- `aws s3 ls`

Removing the named volume would also remove the data managed within that volume.

## 10. Architecture Limitations

This project is intended for local learning and experimentation.

The architecture has several limitations:

- A single SeaweedFS instance is used.
- Primary and backup buckets share the same infrastructure.
- Backup operations are performed manually.
- There is no off-site backup.
- There is no multi-node high availability.
- No production-grade disaster recovery mechanism is implemented.

Therefore, this project demonstrates logical backup and recovery, rather than full disaster recovery.

## 11. Conclusion

CLOUD-02 demonstrates a fundamental object storage architecture using Docker and an S3-compatible API.

The project provides practical experience with storage deployment, authenticated object operations, backup and restoration workflows, persistent volumes, and data integrity verification.
