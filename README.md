# AgentEarth Integrations

Connectors, skills, and plugins for connecting AI agents to AgentEarth.

## Install an integration

- [Hermes](hermes/README.md) · AgentEarth MCP over OAuth and 31 bundled skills.

## Repository layout

Organize integrations by agent first, then by integration type. Each published package lives in its own directory with the files required by the agent's installer.

```text
agentearth-integrations/
├── README.md
├── LICENSE
└── hermes/
    ├── README.md
    └── plugin/
        ├── README.md
        ├── plugin.json
        ├── mcp.json
        └── skills/
            └── <skill-name>/
                └── SKILL.md
```

| Item | Convention |
| --- | --- |
| Agent directory | Use the product name in lowercase, with hyphens between words. Examples include `hermes`, `codex`, and `claude-code`. |
| Integration type | Use `plugin/` or `connector/` for a single package. Use `skills/` or `plugins/` for a collection of independently installable entries. |
| Skill or plugin directory | Use a stable name based on the integration or its purpose, such as `agentearth/` or `tool-discovery/`. Track versions in manifests or release metadata. |
| Documentation | Use lowercase, hyphenated names such as `installation.md`. Keep standard names such as `README.md`, `LICENSE`, and `SKILL.md`. |
| Agent requirements | Preserve any manifest path, hidden directory, filename, or casing the installer requires. These requirements take precedence over the general conventions above. |

Keep bundled skills inside the package that installs them, as in `hermes/plugin/skills/`. Use an agent-level `skills/` directory for skills that are installed individually. Add agent directories and top-level `shared/`, `examples/`, or `docs/` directories when there is content to publish. Keep package files together so installers and reviewers can use the same checkout.

## Add an integration

1. Clone this repository and create a branch from `main`.
2. Add the complete publishable package to the appropriate agent and integration directory. Include its manifests, README, and bundled skills where applicable. Keep credentials and local test files out of the public package.
3. Add or update the agent README with installation and authentication steps. Link it from **Install an integration** above.
4. Validate the package using the agent's installer or validator. Open a pull request to `main` in this repository with the package and its installation instructions.
5. If the agent uses a separate marketplace, follow its submission process once the installation package is published.

### Example: Publishing the Hermes plugin

1. Clone this repository and create a branch.

   ```bash
   git clone https://github.com/agentearth-ai/agentearth-integrations.git
   cd agentearth-integrations
   git switch -c add-hermes-plugin
   ```

2. Copy the contents of `Skills/hermes/plugin/` from AgentEarth-ClientApps into `hermes/plugin/`. Include `plugin.json`, `mcp.json`, the plugin README, and all 31 bundled skills. Link to the package from `hermes/README.md`.
3. Run `hermes plugins validate hermes/plugin --install-deps` and open a pull request to `main` in this repository.
4. After that PR merges, open a separate PR to `NousResearch/hermes-agent` that adds `plugin-catalog/agentearth.yaml`. Set `repo` to this public repository, `subdir` to `hermes/plugin`, and `sha` to the full 40-character commit that contains the reviewed package. See the [Hermes catalog submission guide](https://hermes-agent.nousresearch.com/docs/developer-guide/plugins/catalog-submission).
