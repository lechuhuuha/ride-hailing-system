# Thought Process (Concise)

This note captures key assumptions and trade-offs behind the design in `README.md`.

## Why these sizing numbers?

- `7.5M/day` trips comes from `10M riders * 30% active requesters * 2.5 trips/day`.
- `87 rps` average is `7.5M / 86,400 seconds`.
- `435 rps` peak uses a `5x` busy-hour multiplier.
- Driver location ingest is `200k online drivers / 5s = 40k updates/s`.

## Why `~1KB` trip row and `12 x 0.5KB` events?

- `~1KB` trip row is a conservative planning estimate including schema growth margin.
- `12 events/trip` is a midpoint for lifecycle + dispatch/control events.
- `0.5KB/event` assumes compact event payload + metadata.

## Why Redis for realtime, Postgres for truth?

- Redis handles very high-frequency location + matching reads/writes.
- Postgres stores durable trip lifecycle and audit/event history.
- Periodic snapshots/checkpoints support degraded operation and recovery.

## Why region-owned writes + cross-region sync?

- One region owns each trip write path for strong consistency and simpler correctness.
- Cross-region sync is for DR standby and failover readiness, not multi-region active writes for one trip.

## Why H3 + ETA + batch offers?

- H3 gives fast geographic candidate lookup.
- ETA ranking improves pickup quality vs straight-line distance.
- Batch offers reduce time-to-match; CAS + TTL + cooldown control race/spam side effects.

## Lean index strategy

- Keep only indexes needed by core APIs.
- Avoid heavy global indexes by default in write-heavy tables.
- Add optional indexes only when query metrics prove sustained need.

## Failure posture

- Redis degradation modes: stale snapshot -> limited radius -> DB fallback.
- Geo routing is active-active by nearest healthy region, with failover on regional outage.
