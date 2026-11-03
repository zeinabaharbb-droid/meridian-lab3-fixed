# Meridian Service Desk — Lab 3 fixes (Broken Access Control & Command Injection)

503M Software Security — fixes applied on top of the course's Meridian repo
(https://github.com/mohammad0omar/meridian-servicedesk).

All four fixes are in `src/helpdesk/views/staff.py`. Summary:

1. **A1 — IDOR / object-level authorization.** `view_ticket` (and `export_ticket`)
   now call the existing `ticket_perm_check(request, ticket)`, which raises
   `PermissionDenied` (403) unless the signed-in agent may access the ticket's
   queue. Authentication was present; per-object authorization was missing.

2. **A2 — Vertical privilege escalation.** `promote_user` now requires
   `request.user.is_superuser` and returns 403 "Administrator rights required."
   otherwise. It previously gated on staff, not admin.

3. **A3 — CSRF.** The `@csrf_exempt` decorator was removed from `change_email`,
   so Django's default CSRF-token check applies again; a tokenless POST is 403.

4. **B — Command injection.** `export_ticket` no longer builds a shell command
   from the label. The label is restricted to an allowlist (`[A-Za-z0-9_-]+`)
   and the export is done with plain Python file I/O — no shell is invoked, so
   the label can never be parsed as a command.

See `staff_py_fixes.diff` for the exact changes.
