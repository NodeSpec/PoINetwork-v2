# API Service Tasks

## REQ-001: Persist and retrieve tasks
- [ ] Implement POST /tasks — validate payload, generate id, persist via SQL contract to Primary Database, return created task id
- [ ] Implement GET /tasks/:id — fetch by id from Primary Database, return 404 if not found
- [ ] Verify a task created before a service restart is still retrievable after restart (durability check against Primary Database)

## REQ-002: Expose a REST interface
- [X] Ensure all endpoints accept and return `application/json`
- [X] Add request validation middleware (reject malformed JSON with 400)
- [ ] Add consistent error response shape across endpoints

## Open Dependency
- ⚠ The SQL contract "Task storage queries" to Primary Database has no schema defined yet. Do not implement persistence logic until a schema is proposed and accepted.
