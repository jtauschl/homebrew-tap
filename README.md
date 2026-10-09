<p align="center">
  <img src="img/banner.jpg" alt="A tap filling glass bottles with glowing package cubes on a workshop shelf, next to a green check mark." width="960">
</p>

# jtauschl/tap

Homebrew formulae for my command-line tools, for macOS on Apple Silicon and
Intel. The repository is `jtauschl/homebrew-tap`; Homebrew shortens that to the
tap name `jtauschl/tap`.

## Formulae

| Formula | Description | Source |
| --- | --- | --- |
| [`openproject-ce-mcp`](Formula/openproject-ce-mcp.rb) | MCP server for OpenProject Community Edition | [GitHub](https://github.com/jtauschl/openproject-ce-mcp) · [PyPI](https://pypi.org/project/openproject-ce-mcp/) |

## Install

```bash
brew tap jtauschl/tap
brew trust --formula jtauschl/tap/<formula>
brew install <formula>
```

Homebrew installs a formula from a third-party tap only after you trust it.
`brew trust --formula` trusts that one formula; `brew trust jtauschl/tap`
trusts every formula in this tap, including ones added later.

## Upgrade and uninstall

```bash
brew update
brew upgrade <formula>

brew uninstall <formula>
brew untap jtauschl/tap   # once no formula from this tap is installed
```

## openproject-ce-mcp

```bash
brew tap jtauschl/tap
brew trust --formula jtauschl/tap/openproject-ce-mcp
brew install openproject-ce-mcp
openproject-ce-mcp configure
```

`configure` writes the MCP client configuration; see the project's
[installation guide](https://github.com/jtauschl/openproject-ce-mcp/blob/main/docs/installation.md).
Before uninstalling, remove that configuration first, as described in its
[uninstall section](https://github.com/jtauschl/openproject-ce-mcp/blob/main/docs/installation.md#uninstall).

The formula installs into its own virtual environment on Homebrew's
`python@3.12`, independent of any other Python on the machine.

## How formulae are updated

A formula follows its project's releases. `main` is what `brew update`
delivers, so a formula is tested before it lands there:

1. On a branch, bump the formula's `url` and `sha256` to the new release;
   for a Python formula, regenerate its `resource` blocks (below).
2. Run the **Test formulae** workflow on that branch (Actions, Run workflow,
   pick the branch and the formula name). It styles and audits the formula,
   installs it from source on Apple Silicon and Intel, runs its test block
   and, for a formula published on PyPI, installs the previous release and
   upgrades it on Apple Silicon.
3. Fast-forward `main` to the branch once it passes.

A push to `main` that changes a formula or the CI runs the same workflow
again, for the formulae it touched (all of them when the CI changed).

Another repository's release pipeline can run the same tests before it pushes
a formula, by calling the workflow:

```yaml
jobs:
  test-formula:
    uses: jtauschl/homebrew-tap/.github/workflows/test-formulae.yml@main
    with:
      formula: <formula>
      formula-artifact: <artifact holding the rendered <formula>.rb>
```

### Python formulae

The `resource` blocks are generated, never edited by hand:

```bash
brew tap-new jtauschl/local --no-git
cp Formula/<formula>.rb "$(brew --repo jtauschl/local)/Formula/"
brew trust jtauschl/local
brew update-python-resources --ignore-main-package-cooldown \
  --exclude-packages=pywin32 jtauschl/local/<formula>
cp "$(brew --repo jtauschl/local)/Formula/<formula>.rb" Formula/
```

- Homebrew loads a formula only from a trusted tap, so the formula is copied
  into a local one first.
- Homebrew resolves only packages that have been on PyPI for a day.
  `--ignore-main-package-cooldown` lifts that for the formula's own, just
  released package; every dependency keeps the cooldown.
- `pywin32` is a Windows-only dependency and has no place in a macOS formula.

## Why a tap and not Homebrew Core

Homebrew Core requires a tool to be widely used and maintained to Core's
cadence and review. A tap gives a native `brew install` today, with releases
under the project's control, while PyPI and GitHub stay the canonical sources.
