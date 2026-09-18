# homebrew-tap

A dedicated Homebrew tap for [openproject-ce-mcp](https://github.com/jtauschl/openproject-ce-mcp), an MCP server for OpenProject Community Edition.

## Install

```bash
brew tap jtauschl/tap
brew install openproject-ce-mcp
```

## Upgrade

```bash
brew upgrade openproject-ce-mcp
```

## Uninstall

```bash
brew uninstall openproject-ce-mcp
brew untap jtauschl/tap  # if you don't use any other formula from this tap
```

If you previously ran `openproject-ce-mcp configure`, remove the generated
client configuration first — see the [main project's installation
docs](https://github.com/jtauschl/openproject-ce-mcp/blob/main/docs/installation.md#uninstall)
for the exact steps.

## Verify

```bash
openproject-ce-mcp --version
openproject-ce-mcp configure --help
openproject-ce-mcp doctor --help
```

## Why a dedicated tap, not Homebrew Core

Homebrew Core has strict maintenance and notability requirements: a formula
must be widely used, and the maintainer commits to Core's own release cadence
and heavier review/CI process. A dedicated tap is the standard bridge most
PyPI-distributed CLI tools use before, if ever, graduating to Core — it
provides a native `brew install` path today on both Apple Silicon and Intel
macOS, with full control over release cadence and Formula content, while
[PyPI](https://pypi.org/project/openproject-ce-mcp/) stays the canonical
publication source. A Core submission is worth revisiting only once real,
sustained external usage justifies Core's heavier maintenance commitment.

## Formula maintenance

The Formula's `resource` blocks are machine-generated via Homebrew's own
`brew update-python-resources`, following the actual PyPI dependency
resolution at generation time — not a copy of this project's own `uv.lock`,
which pins exact versions for its own CI. Never hand-edit the `resource`
blocks; regenerate them instead:

```bash
brew update-python-resources \
  --version=<new version> \
  --package-name=openproject-ce-mcp \
  --exclude-packages=pywin32 \
  Formula/openproject-ce-mcp.rb
```

`pywin32` is excluded deliberately: it's a Windows-only dependency of `mcp`
(platform-marker-gated) and has no place in a macOS Formula.

A pull request against this Formula is opened automatically after each
`openproject-ce-mcp` PyPI release (see that project's `publish.yml`). Review
the diff before merging — this bumps `url`/`sha256` and regenerates every
`resource` block, so a passing CI run on both architectures (see below)
before merge is the real safety net, not a manual read of 28 hashes.

## CI

`.github/workflows/test-formula.yml` verifies a real `brew install` from this
tap on both Apple Silicon (`macos-latest`) and Intel (`macos-15-intel`) macOS,
runs `--version`/`configure --help`/`doctor --help`, and exercises the
upgrade and uninstall flows — gated to pull requests and manual dispatch
only, never a direct push to `main`, matching this tap's one real cost
concern (macOS CI minutes).
