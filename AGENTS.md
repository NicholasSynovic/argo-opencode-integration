# AGENTS.md

- Launcher: `argonne-claude.sh`.
- Proxy entrypoint: `main.py`.
- `argonne-claude.sh` only parses `--backend` and `--identity`; every other CLI arg is ignored.
- `--backend` is required and must be `argo` or `asksage`.
- Argo uses an SSH tunnel to `apps.inside.anl.gov:443` through `homes.cels.anl.gov`, then the local proxy on `127.0.0.1:8083`.
- On Aurora UANs, `homes.cels.anl.gov` needs `ProxyJump logins.cels.anl.gov` in `~/.ssh/config`; on compute nodes the launcher auto-adds the UAN + `logins.cels.anl.gov` hop when `$PBS_JOBID` is set.
- Argo startup expects `python3.12` and `aiohttp`.
- AskSage resolves the key from `--identity`, `ASKSAGE_API_KEY`, or `ASKSAGE_TOKEN_FILE` (`~/.asksage/token` by default).
- AskSage uses `ASKSAGE_BASE_URL` for the Anthropic endpoint and sets `NODE_EXTRA_CA_CERTS` to `certs/incommon-rsa-server-ca-2.pem` unless already set.
- AskSage model discovery and adaptive-thinking probing are built into the launcher; `ASKSAGE_MODEL`, `ASKSAGE_SMALL_FAST_MODEL`, and `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING` short-circuit that logic.
- `make build` tags the package from the latest git tag, builds `dist/`, then installs the tarball with `uv`.
- `make create-dev` runs `pre-commit install`, `pre-commit autoupdate`, removes `env/`, then `uv sync`.
- Fast checks: `./argonne-claude.sh --backend=asksage`; `python3.12 main.py` for the proxy.
