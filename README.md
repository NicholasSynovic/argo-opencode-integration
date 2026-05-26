# argo-opencode-integration

Launch Claude Code against Argonne-hosted backends.

This repo has two pieces:
- `argonne-claude.sh` launches Claude Code with the right backend env vars.
- `main.py` is the local proxy used by the `argo` backend.

Supported backends:
- `argo`: tunnels to Argonne's internal Argo API through `homes.cels.anl.gov`.
- `asksage`: talks directly to AskSage's Anthropic-compatible endpoint.

## Install

Prereqs:
- Python 3.14+
- `uv`
- Claude Code

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Clone the repo somewhere stable and add it to your `PATH`:

```bash
git clone https://github.com/NicholasSynovic/argo-opencode-integration.git ~/argo-opencode-integration
export PATH="$HOME/argo-opencode-integration:$PATH"
```

Optional dev setup:

```bash
make create-dev
```

Build the package:

```bash
make build
```

## Run

`--backend` is required.

Argo:

```bash
argonne-claude.sh --backend=argo
```

AskSage:

```bash
argonne-claude.sh --backend=asksage
```

Override the identity if needed:

```bash
argonne-claude.sh --backend=argo --identity=jdoe
argonne-claude.sh --backend=asksage --identity=sk-asksage-...
```

For Argo, `--identity` sets the SSH username for the target and jump hosts.
If omitted, it defaults to your current local username.

## Architecture

Argo flow:
1. SSH tunnels local port `8082` to `apps.inside.anl.gov:443` via `homes.cels.anl.gov`.
2. `main.py` listens on `127.0.0.1:8083` and rewrites requests for the Argo API.
3. Claude Code talks to `http://127.0.0.1:8083/argoapi/`.

AskSage flow:
1. The launcher resolves an API key from `--identity`, `ASKSAGE_API_KEY`, or `~/.asksage/token`.
2. It queries `${ASKSAGE_BASE_URL}/v1/models` and probes adaptive-thinking support unless you override that behavior with `ASKSAGE_MODEL`, `ASKSAGE_SMALL_FAST_MODEL`, or `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING`.
3. It sets `NODE_EXTRA_CA_CERTS` to `certs/incommon-rsa-server-ca-2.pem` unless you already exported one.

## Aurora notes

UANs need this SSH config so `homes.cels.anl.gov` routes through the login host:

```sshconfig
Host homes.cels.anl.gov
    ProxyJump logins.cels.anl.gov
```

On compute nodes, the launcher adds the extra UAN hop automatically when `$PBS_JOBID` is set.

## AskSage token file

Recommended setup:

```bash
mkdir -p ~/.asksage && chmod 700 ~/.asksage
printf '%s\n' 'your-asksage-key' > ~/.asksage/token
chmod 600 ~/.asksage/token
```

## Useful overrides

- `CLAUDE_EXECUTABLE` changes the `claude` binary.
- `ARGO_AURORA_UAN` and `ARGO_SSH_JUMP` override Aurora jump-host selection.
- `ASKSAGE_BASE_URL` targets another AskSage tenant.
- `ASKSAGE_MODEL` and `ASKSAGE_SMALL_FAST_MODEL` pin models and skip discovery.

## Debugging Argo

Run the proxy directly:

```bash
python3.14 main.py
```

Run the tunnel by hand:

```bash
ssh -L 8082:apps.inside.anl.gov:443 -N homes.cels.anl.gov
```
