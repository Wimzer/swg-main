# Planetary Mining Droid Checkpoint - 2026-08-28

This checkpoint records the complete Planetary Mining Droid source state after
the durable account-ledger deployment and one-worker optimization. It is a
project checkpoint, not proof of later deployment or gameplay behavior.

## Source Identity

- Superproject branch: `feature/planetary-mining-droid-ledger`
- `src` branch: `feature/planetary-mining-droid-ledger`
- `src` commit: `c99df2fb` (`Finalize planetary mining ledger integration`)
- `dsrc` branch: `feature/planetary-mining-droid-ledger`
- `dsrc` ledger commit: `6ab9b3751` (`Finalize planetary mining ledger integration`)
- `dsrc` client checkpoint commit: `21435e4b5` (`Archive planetary mining client revision 35`)
- The source branches retain the earlier PMD feature history.
- Every path changed in the deployed ledger target snapshots byte-matched the
  corresponding final branch path before commit.
- Generated `dsrc/build/` output is not committed.

## Implemented State

- PMD survey, crafting, region, launch, return, yield, expertise inheritance,
  login recovery, and loose-client support are retained from the feature
  history.
- Oracle migration 272 creates `PLANETARY_MINING_JOBS` and
  `PMD_JOBS_ACCOUNT_IDX`.
- `PLANETARY_MINING_JOB_API` serializes reservations per station account,
  validates character ownership, enforces three jobs per account and galaxy,
  handles exact-sequence retries idempotently, and releases completed jobs.
- GameServer and DatabaseServer exchange `PlanetaryMiningJobRequest` and
  `PlanetaryMiningJobResponse` messages.
- DatabaseServer services PMD ledger operations through one dedicated
  `DB::TaskQueue` worker, preserving isolation while using one worker thread and
  one Oracle session.
- Java launch, release, retry, and login reconciliation use the Oracle ledger;
  the earlier PMD AccountFeature reservation path is no longer used.
- Complete loose client revision 35 is archived at
  `dsrc/client files/planetary_mining_droid_35`. Its portable SHA-256 manifest
  verifies all 12 installable payload files. PMD remains a loose-file client
  patch and is not packaged as a TRE.

## Verification

- The deployment target successfully completed `compile_src` and
  `compile_java`; deployed Java classes were major version 55.
- A fresh local `ant compile_java` compiled 5,635 source files successfully
  with Java 11 source/target output. Checked PMD-related classes are major
  version 55.
- The final `src` staged diff passed `git diff --check`.
- Existing CRLF/trailing-whitespace formatting in `base_class.java` and the
  captured `ui/ui_styles.inc` was preserved rather than normalized.
- The deployed Oracle schema reported version `272/272`, valid ledger table,
  index, package specification, and package body, with no package errors.
- The deployed object-template registry contained 60,102 entries and all four
  PMD server templates.
- User-reported testing covered PMD return, resource yield, expertise
  inheritance, the three-job account limit, login-time Survey License recovery,
  and persistence through a managed restart.

## Deployment Record

- Initial deployment backup:
  `/home/swg/backups/pmd-ledger-20260827T231647Z/predeploy.tgz`
- Initial backup SHA-256:
  `49911800f7541d519bda0ab2ee113f74665edbf8ffdddbc8ffb85b01728421a7`
- One-worker backup:
  `/home/swg/backups/pmd-one-worker-20260828T015149Z`
- One-worker predeployment manifest SHA-256:
  `a14c2c2138267455d9b1556b017df337c665bef27f6ee720520747f5d6844bcd`
- Deployed `SwgGameServer` SHA-256:
  `f8c00871fd3a50dbc3163995ae4131e8611c71003eaa9846de9c020bc552d67d`
- Deployed one-worker `SwgDatabaseServer` SHA-256:
  `2c138649d94134a2771284844895ebdd172b9184f3a7299dae7167aab0a101c5`
- The one-worker change reduced DatabaseServer measurements from 11 to 10
  threads and from 10 to 9 Oracle TCP sessions. Measured RSS was effectively
  unchanged.
- Stable post-restart checks found one each of TaskManager, LoginServer,
  CentralServer, ConnectionServer, and SwgDatabaseServer, plus 38 PlanetServers
  and 38 SwgGameServers.

## Load Simulation

- An approved live SQL transaction inserted 100 synthetic account/character
  pairs and called `PLANETARY_MINING_JOB_API.RESERVE_JOB` serially 100 times to
  model an already-queued one-worker burst.
- All 100 reservations succeeded and all test data was rolled back. Independent
  postflight found zero synthetic residue and zero ledger rows.
- Total measured package batch time was 2,272.756 ms. Service latency was
  0.363 ms p50, 0.474 ms p95, 5.812 ms p99, and 2,229.933 ms maximum.
- The unexplained 2,229.933 ms outlier dominated the batch. The measured
  43.999 requests/second must not be treated as established steady-state
  throughput.
- This simulation did not traverse real clients, GameServer messaging, Java
  callbacks, or the C++ task queue and did not test same-account contention.

## Known Limits And Follow-Up

- Migration 272 creates an empty ledger and has no historical backfill. Login
  reconciliation reserves rows for character-owned active jobs after login.
- A focused launch, fourth-slot rejection, and return test remains desirable on
  the one-worker binary.
- Concurrent same-account reservations, account transfer, forced-crash
  recovery, 2,000 active rows, and a true five-launches-per-second workload are
  not yet tested.
- Readiness arrived around 3 minutes 20 seconds, after the bounded three-minute
  poll ended; a subsequent bounded read captured the ready marker.
- No live reset, deployment, restart, database write, or hard-mode operation is
  part of this source checkpoint.
