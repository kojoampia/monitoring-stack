# Session Summary: Monitoring Stack Stabilization

Date: 2026-03-22
Workspace: /home/kojo/work/infra/0-monitoring-stack

## Executive Summary
This session focused on stabilizing the observability stack by resolving:
- MinIO setup connectivity failures from helper and dependent services.
- Loki ingestion failures caused by unlabeled streams.
- Excessive Alloy parsing/noise logs and Tempo scheduler-noise ingestion.

The stack is now in a healthy state with:
- Reliable MinIO bootstrap and bucket creation.
- Clean Loki ingestion path with required labels.
- Reduced log noise in Alloy and Tempo-derived log streams.

## Problems Addressed

### 1) MinIO alias setup repeatedly timed out
Symptoms:
- minio-setup repeatedly logged:
  - Unable to initialize alias
  - dial tcp 172.27.0.2:9000: i/o timeout

Findings:
- DNS resolution inside containers was correct.
- MinIO container was healthy.
- Direct TCP from peer containers to MinIO intermittently timed out in this environment.

### 2) Loki rejected incoming logs
Symptoms:
- write operation failed
- error at least one label pair is required per stream

Findings:
- Sender was Alloy (loki.write.local_loki).
- Streams were reaching Loki without guaranteed labels.

### 3) Excessive parsing/noise logs
Symptoms:
- Alloy: could not extract timestamp, skipping line
- Tempo logs included repetitive scheduler no-jobs messages that were informational but noisy.

Findings:
- Timestamp parse noise concentrated around alloy and minio container logs.
- Tempo backend scheduler no-jobs lines were expected in this setup and not a functional failure.

## Files Changed (Final)

### docker-compose.yml
Final persistent changes:
- Added MinIO readiness healthcheck.
- Updated minio-setup to one-shot semantics and robust startup sequence:
  - restart policy changed to on-failure.
  - depends_on now waits for MinIO health.
  - network_mode set to service:minio.
  - alias command uses localhost endpoint (127.0.0.1:9000) from shared namespace.
- Added/normalized service restart and host/domain settings for several services.
- Ensured dependent services (loki/mimir/tempo) wait for minio-setup completion.

Impact:
- minio-setup now completes and exits 0 reliably.
- Buckets are created automatically before dependent services proceed.

### configs/alloy-config.alloy
Final persistent changes:
- Added discovery.relabel docker_logs pipeline.
- Added target drop rules for alloy and minio log scraping to reduce timestamp parse noise.
- Added deterministic labeling path:
  - ensures job label is set to docker.
  - maps Docker container name into container label.
- Inserted log processing stage to drop repetitive Tempo scheduler-noise messages:
  - BackendScheduler/Next
  - no jobs found
  - error calling scheduler
- Routed log flow:
  - loki.source.docker -> loki.process.drop_tempo_noise -> loki.relabel.ensure_labels -> loki.write.local_loki

Impact:
- Loki no longer receives unlabeled streams.
- Parsing/noise logs reduced significantly.
- Tempo scheduler informational chatter is filtered from ingestion.

### .env
Final state:
- S3 endpoint remains minio:9000.
- Temporary endpoint experiments were reverted.

Impact:
- Consistent in-network service addressing retained.

## Validation Performed

### MinIO bootstrap and bucket creation
Verified:
- minio container reports healthy.
- minio-setup runs and exits 0.
- Buckets created:
  - loki
  - mimir
  - mimir-alertmanager
  - tempo

### Loki ingestion health
Verified:
- No recurring unlabeled-stream 400 errors after Alloy relabeling update.
- A transient old-timestamp rejection occurred during replay window and then stopped.

### Alloy pipeline health
Verified:
- Alloy starts with updated config.
- No recurring parser startup errors in final state.
- No recurring status=400 push errors in final checks.

### Tempo runtime health
Verified:
- Tempo starts and remains up.
- No functional outage found; scheduler no-jobs lines categorized as expected background behavior for this setup.

## Attempted and Reverted Adjustments

The following were attempted during diagnosis but intentionally reverted because they were unnecessary or environment-specific:
- Host-routed S3 endpoint experiments (host.docker.internal and gateway/port mappings).
- Temporary MinIO host port remapping attempts.
- Tempo target override using internal module list (unsupported by installed Tempo module resolver).

Final configuration does not include these temporary experiments.

## Operational Notes

- If host-level Docker bridge forwarding constraints reappear, MinIO bootstrap remains resilient because minio-setup shares MinIO network namespace.
- Tempo scheduler no-jobs entries are now filtered at ingestion rather than changing Tempo runtime modules.
- Alloy excludes alloy/minio logs from Docker tail ingestion to avoid known timestamp parse noise.

## Final Outcome
All requested stabilization work for this session was completed:
- MinIO setup reliability fixed.
- Loki ingestion correctness fixed.
- Alloy/Tempo noise reduced while preserving core telemetry flow.
