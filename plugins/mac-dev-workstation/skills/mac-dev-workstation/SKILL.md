---
name: mac-dev-workstation
description: Bootstrap a fresh macOS development workstation when the request is to recreate or repair a new Mac setup with Homebrew, shell PATHs, nvm/Node, Python, Rust, Ollama, OrbStack, Slack, and standard CLI tooling.
---

# Mac Dev Workstation

Use this skill when a Mac has just been formatted, a dev environment needs to be rebuilt from scratch, or the user wants the standard local workstation stack restored.

Keep the workflow focused on the pieces that were actually part of the setup:

- Homebrew as the package manager
- shell PATH setup for `~/.local/bin`, `~/.cargo/bin`, and `nvm`
- Node installed with `nvm` and set to the current LTS
- Python installed from Homebrew
- Rust installed with `rustup`
- Ollama installed as both CLI/server and desktop app when the UI is requested
- OrbStack installed for local containers
- Slack and other common GUI apps when asked

Prefer the least disruptive path:

- do not replace an existing working install unless the user asks
- keep shell changes in the appropriate startup files
- verify the result with version checks before claiming success

Use macOS System Settings for keyboard remaps and similar GUI-only preferences; this skill does not try to automate those settings unless the user explicitly asks for a scripted workaround.
