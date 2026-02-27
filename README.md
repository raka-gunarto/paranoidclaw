# ParanoidClaw

My own hardened OpenClaw setup with monitoring. Main features include:

- [agent-panopticon](https://github.com/raka-gunarto/agent-panopticon): `mitmproxy` bidirectional proxy, enforced by iptables
    - Secret substitution
    - Request logging
    - Outbound traffic filtering
- Run OpenClaw container with high UID
    - Near-zero permissions on actual host
- Container hardening
    - no-new-privileges: true (no suid binaries allowed, etc. priv esc is hard)
    - cap-drop: ALL (we don't need any)
- cgroups resource limits
- Using bwrap with user namespaces for unprivileged sandboxing [(PR Pending Merge)](https://github.com/openclaw/openclaw/pull/28599)

## Quickstart

### Setup

1. Clone the repository:
    ```bash
    git clone https://github.com/raka-gunarto/paranoidclaw.git
    cd paranoidclaw
    ```
2. Clone OpenClaw with bwrap support (will update submodule if/when PR gets merged):
    ```bash
    git clone --branch feat/sandbox-bwrap https://github.com/raka-gunarto/openclaw-bwrap.git
    ```
3. Build the base container image tagged as `paranoidclaw-openclaw-base`:
    ```bash
    docker build -t paranoidclaw-openclaw-base \
      --build-arg OPENCLAW_DOCKER_APT_PACKAGES="bubblewrap" \
      ./openclaw-bwrap
    ```
4. Configure proxy secrets in `.env.proxy`:
    ```bash
    cp .env.proxy.example .env.proxy
    # Edit with your secret values
    ```
5. Configure the proxy allowlist `allowlist.txt` configuration to permit your required external domains.
6. Load the apparmor profile (if your LSM is apparmor):
    ```
    sudo apparmor_parser -a openclaw-apparmor
    ```
7. Start the services:
    ```bash
    docker compose up
    ```
8. Configure OpenClaw with the proxy placeholder key secrets from your `.env.proxy` file and `bwrap` sandbox backend.

## Questions you may ask

### Why?
Fully autonomous agents with shell and network access that read input from external channels forms [The Lethal Trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/). I want to play around with these tools and my machines have other things running on them.

And incidents like CVE-2026-25253.

This was also a fun way to learn more about Linux hardening.

### Is this fully safe?
Nothing is fully safe, but this gives me enough comfort while experimenting. In fact you could probably harden even further with things like gVisor.

This setup still doesn't protect you from your LLM hallucinating and nuking your entire mailbox if you gave it access to do so.

The main purpose of this setup is to: 
- reduce data exfiltration risk by limiting network access and credential visibility to the LLM.
- limit blast radius of LLM hallucination / prompt injection with some hardening.

### Why did you write your own / extend mitmproxy for the bidirectional proxy?
Initially I used other combinations of ingress / egress proxies, but I thought this was simpler and I thought it'd be fun.