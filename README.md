# agent-marketplace

Mein Plugin-Marktplatz für AI Coding Agents — MCP Server, Skills, Commands, Hooks. Eine Registry, alle Plugins von [@Seretos](https://github.com/Seretos).

## Install ein Plugin

**Claude Code:**

```
/plugin marketplace add Seretos/agent-marketplace
/plugin install <plugin-name>@agent-marketplace
```

Verfügbare Plugins stehen in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).

## Wie neue Plugins reinkommen

Plugin-Repo released → schickt `repository_dispatch` Event → CI öffnet PR → ich mergsts → Eintrag ist live.
