# Gotchas

Observed failures only. Add a line when the skill misfires, with the retro or issue that saw it.

- `git merge --ff-only` on the host fails after any local edit there, including one made by
  another automation user sharing the checkout. The deploy stops; clean by hand and rerun.
- The sudoers path and the command in the caller must match byte for byte; a trailing
  argument that sudoers does not list gives "a password is required".
