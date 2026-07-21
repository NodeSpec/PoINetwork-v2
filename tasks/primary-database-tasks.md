# Primary Database Tasks

## REQ-001: Persist and retrieve tasks
- [ ] Define a `tasks` table/schema (id, fields TBD) matching what API Service needs for POST /tasks
- [ ] Ensure storage is durable across process/container restarts (persistent volume, not in-memory)
- [ ] Add index on task id for GET /tasks/:id lookups

## Open Dependency
- ⚠ This node's schema artifact should be updated once the "Task storage queries" SQL contract's schema is proposed and accepted — do not finalize table structure before that.
