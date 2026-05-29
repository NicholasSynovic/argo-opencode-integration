# AGENTS.md

- Entrypoints: `./argonne-claude.sh` launches Claude Code; `python3.14 main.py` runs the local proxy.
- `argonne-claude.sh` only consumes `--backend` and `--identity`; all other CLI args are ignored.
- `--backend` is required and must be `argo` or `asksage`.
- Argo currently SSHes to `homes-gce`, forwards local port `8082` to `apps.inside.anl.gov:443`, and the proxy listens on `127.0.0.1:8083`.
- On Aurora compute nodes (`$PBS_JOBID` set), the launcher prepends the UAN from `$PBS_O_HOST` or `ARGO_AURORA_UAN` and `logins.cels.anl.gov` to `ARGO_SSH_JUMP` automatically.
- For Aurora UANs, `homes.cels.anl.gov` needs `ProxyJump logins.cels.anl.gov` in `~/.ssh/config`.
- `CLAUDE_EXECUTABLE` overrides the Claude binary.
- AskSage key precedence is `--identity` > `ASKSAGE_API_KEY` > `ASKSAGE_TOKEN_FILE` (`~/.asksage/token` by default).
- AskSage uses `ASKSAGE_BASE_URL` (default `https://api.asksage.anl.gov/server/anthropic`) and sets `NODE_EXTRA_CA_CERTS` to `certs/incommon-rsa-server-ca-2.pem` unless already set.
- AskSage model discovery and adaptive-thinking probing are built into the launcher; `ASKSAGE_MODEL`, `ASKSAGE_SMALL_FAST_MODEL`, and `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` short-circuit that logic.
- `make build` derives the package version from the newest git tag, builds `dist/`, then installs the tarball with `uv`.
- `make create-dev` runs `pre-commit install`, `pre-commit autoupdate`, removes `env/`, then `uv sync`.
