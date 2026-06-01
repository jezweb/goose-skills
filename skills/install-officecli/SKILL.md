---
name: install-officecli
version: 1.0.0
description: "Install the OfficeCLI binary (create/read/edit .docx/.xlsx/.pptx with no Microsoft Office) and register it as a Goose MCP server. Use when the user wants Office-document capabilities, asks to 'install officecli' / 'add Word/Excel/PowerPoint support', or a skill/app needs to generate Office files."
metadata:
  category: "setup"
  homepage: "https://github.com/iOfficeAI/OfficeCLI"
  license-of-tool: "Apache-2.0"
  installs-bin: "officecli"
---

# Install OfficeCLI

Installs the [OfficeCLI](https://github.com/iOfficeAI/OfficeCLI) single binary (Apache-2.0,
pure-format, no Microsoft Office required, cross-platform) and registers it as a Goose MCP
server. After this, the agent can create/read/edit Word, Excel and PowerPoint files — with
real formula evaluation, template merge, a validation linter, and PNG/HTML rendering.

**Safety rule (do not skip):** never `curl | bash` a release blindly. Download the pinned
release binary, **verify it against the published `SHA256SUMS`**, and only then run it. The
checksum step is the point of this skill.

## Steps

Pinned version (bump deliberately; always re-verify the checksum):

```bash
VER="v1.0.102"
REPO="iOfficeAI/OfficeCLI"

# 1. Detect platform → release asset name
os=$(uname -s); arch=$(uname -m)
case "$os/$arch" in
  Darwin/arm64) asset="officecli-mac-arm64" ;;
  Darwin/x86_64) asset="officecli-mac-x64" ;;
  Linux/x86_64|Linux/amd64) asset="officecli-linux-x64" ;;
  Linux/aarch64|Linux/arm64) asset="officecli-linux-arm64" ;;
  *) echo "Unsupported platform $os/$arch — see $REPO/releases"; exit 1 ;;
esac

# 2. Download the binary + the checksums file
dir=$(mktemp -d); cd "$dir"
base="https://github.com/$REPO/releases/download/$VER"
curl -fsSL -o officecli "$base/$asset"
curl -fsSL -o SHA256SUMS "$base/SHA256SUMS"

# 3. VERIFY checksum — abort if it doesn't match
want=$(grep " $asset\$" SHA256SUMS | awk '{print $1}')
got=$(shasum -a 256 officecli | awk '{print $1}')
[ -n "$want" ] && [ "$want" = "$got" ] || { echo "CHECKSUM MISMATCH — aborting (want=$want got=$got)"; exit 1; }
echo "checksum ok: $got"

# 4. Install onto PATH
mkdir -p "$HOME/.local/bin"
mv officecli "$HOME/.local/bin/officecli"
chmod +x "$HOME/.local/bin/officecli"
[ "$os" = "Darwin" ] && xattr -d com.apple.quarantine "$HOME/.local/bin/officecli" 2>/dev/null || true

# 5. Verify it runs
"$HOME/.local/bin/officecli" --version
```

If `~/.local/bin` is not on `PATH`, add it (e.g. append `export PATH="$HOME/.local/bin:$PATH"`
to the shell profile) so Goose's spawned MCP process can find it.

## Register as a Goose MCP server

OfficeCLI ships an MCP stdio server (`officecli mcp`). Goose has no `mcp add` for it, so add the
extension to `~/.config/goose/config.yaml` under `extensions:` (mirror how the other stdio
extensions there are written — do not duplicate an existing entry):

```yaml
  office-docs:
    enabled: true
    type: stdio
    cmd: officecli          # or the absolute ~/.local/bin/officecli if not on PATH
    args: ["mcp"]
    timeout: 60
    name: office-docs
```

Restart Goose (or reload extensions) so it picks up the new server. The tool appears as a single
`officecli` gateway tool with a `command` enum (create/view/get/set/add/validate/batch/merge/
load_skill/…) — call `officecli` with `command: help, format: xlsx` to see the element schema.

## Verify end-to-end

```bash
t=$(mktemp -d)
officecli create "$t/x.xlsx" --force
printf 'Item,Amount\nA,100\nB,250\n' > "$t/d.csv"
officecli import "$t/x.xlsx" /Sheet1 "$t/d.csv" --header
officecli set "$t/x.xlsx" /Sheet1/B4 --prop formula="SUM(B2:B3)"
officecli view "$t/x.xlsx" text   # B4 should show 350 — formula evaluated, no Excel needed
```

If that prints `350`, OfficeCLI is working. For nicer output, the agent can call
`officecli` `command: load_skill` (excel / word / pptx / financial-model / pitch-deck) to pull
in OfficeCLI's own quality guidance before generating a document.

## Notes
- Pure-format via bundled libraries — works on a headless machine with no Office installed.
- The binary is ~32 MB; install once per machine.
- To update: bump `VER`, re-run (the checksum step re-verifies the new release).
- Uninstall: `rm ~/.local/bin/officecli` and remove the `office-docs` block from `config.yaml`.
