# Gotchas

Observed failures only. Add a line when the skill misfires, with the retro or issue that saw it.

- A tag ruleset written as `refs/tags/v*` blocks the force-move of the floating `v1`. Use
  `refs/tags/v*.*.*` for the immutable tags and leave `v1` out.
