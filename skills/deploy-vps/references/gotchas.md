# Gotchas

Observed failures only. Add a line when the skill misfires, with the retro or issue that saw it.

- `git merge --ff-only` on the host fails after any local edit there, including one made by
  another automation user sharing the checkout. The deploy stops; clean by hand and rerun.
- The sudoers path and the command in the caller must match byte for byte; a trailing
  argument that sudoers does not list gives "a password is required".
- `sudo -u SERVICE_USER` has no login session: `systemctl --user` fails with "Failed to
  connect to user scope bus" and an apply script reports every service as "did not start"
  while touching none. The deploy script must export `XDG_RUNTIME_DIR=/run/user/<uid>` and
  `DBUS_SESSION_BUS_ADDRESS=unix:path=$XDG_RUNTIME_DIR/bus` first (first CI deploy, 2026-11-12).
- "Up to date" must mean applied, not pulled. Comparing `HEAD` with `origin/main` lets a manual
  `git pull` on the host make CI skip the apply forever. Keep a marker with the last commit
  the apply finished on and compare against that (2026-11-12).
- A federated credential created before its tag existed cannot be edited to add the tag;
  the runner gets `403 calling actor does not have enough permissions` at auth-key creation.
  Create the tag in the policy first, then the credential (2026-11-12).
- The deploy script is itself under version control, and bash reads a script incrementally:
  a run that fast-forwards over a change to the script keeps executing the OLD copy. After
  the fast-forward, `exec "$0" "$@"` so the new version runs; the second pass sees HEAD at
  the target and does not merge again (2026-11-12, third CI deploy).
