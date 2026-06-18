# Claude Code — Configuración del harness

Config sincronizada entre mis 2 equipos. Solo se versiona la **configuración**; sesiones, caché, secretos y binarios de plugins están en `.gitignore` (se regeneran/reinstalan por máquina).

## Plugins (reinstalar en equipo nuevo)

Los binarios de `plugins/` NO se suben (148M, regenerables). Reinstalar desde sus marketplaces:

| Plugin | Repo (marketplace) | Qué aporta |
|--------|--------------------|------------|
| **caveman** | `JuliusBrussee/caveman` | Modo salida ultra-comprimido (menos tokens) |
| **ponytail** | `DietrichGebert/ponytail` | Modo "lazy": solución mínima que funciona |
| **engram** | `Gentleman-Programming/engram` | Memoria persistente (MCP `mem_*`) |
| **everything-claude-code** | `affaan-m/everything-claude-code` | 14+ agents, 56+ skills, 33+ commands, hooks |

Reinstalar (dentro de Claude Code):
```
/plugin marketplace add JuliusBrussee/caveman
/plugin marketplace add DietrichGebert/ponytail
/plugin marketplace add Gentleman-Programming/engram
/plugin install caveman ponytail engram
```
`enabledPlugins` ya queda en `settings.json`.

## Herramientas externas (no son plugins)

| Tool | Instalación | Para qué |
|------|-------------|----------|
| **gentle-ai** (1.40.2) | `brew tap Gentleman-Programming/homebrew-tap && brew install gentle-ai` | Stack SDD + orquestación + binario Engram. Luego `gentle-ai sync`. El tap requiere `brew trust gentleman-programming/tap` |
| **engram** (1.16.3) | viene con gentle-ai (brew) | Servidor MCP de memoria |

## Servidores MCP (`mcp/`)

- **engram** → `engram mcp --tools=agent` (memoria persistente)
- **context7** → docs de librerías al día
- **notebooklm** → research sobre notebooks

> El registro del MCP vive en `~/.claude.json` (fuera de este repo, contiene auth). En equipo nuevo: registrar `engram` en `mcpServers` apuntando al binario de brew, o correr `gentle-ai sync`.

## Contenido versionado

- `skills/` (69) · `commands/` (70) · `agents/` (45) · `rules/` (65 .md)
- `CLAUDE.md` — instrucciones globales (Engram, SDD, personas, overrides)
- `settings.json` — model `opus`, theme `dark`, plugins habilitados
- `output-styles/`, `keybindings.json`, `marketplace.json`

## NO versionado (`.gitignore`)

Secretos (`.credentials.json`, `history.jsonl`), sesiones (`projects/`, `sessions/`), caché, `backups/`, y `plugins/` (148M, reinstalable).

## Setup en equipo nuevo

1. `git clone <repo> ~/.claude`
2. Instalar gentle-ai por brew (ver arriba) + `gentle-ai sync`
3. Reinstalar plugins (ver arriba)
4. Registrar MCPs en `~/.claude.json` (no incluido por seguridad)
5. **Solo WSL** — fix interop permanente:
   ```bash
   echo ':WSLInterop:M::MZ::/init:PF' | sudo tee /etc/binfmt.d/WSLInterop.conf
   sudo systemctl restart systemd-binfmt
   ```

---

# Notas de ECC (everything-claude-code)

### Plugin Manifest Gotchas

If you plan to edit `.claude-plugin/plugin.json`, be aware that the Claude plugin validator enforces several **undocumented but strict constraints** that can cause installs to fail with vague errors (for example, `agents: Invalid input`). In particular, component fields must be arrays, `agents` must use explicit file paths rather than directories, and a `version` field is required for reliable validation and installation.

These constraints are not obvious from public examples and have caused repeated installation failures in the past. They are documented in detail in `.claude-plugin/PLUGIN_SCHEMA_NOTES.md`, which should be reviewed before making any changes to the plugin manifest.

### Custom Endpoints and Gateways

ECC does not override Claude Code transport settings. If Claude Code is configured to run through an official LLM gateway or a compatible custom endpoint, the plugin continues to work because hooks, commands, and skills execute locally after the CLI starts successfully.

Use Claude Code's own environment/configuration for transport selection, for example:

```bash
export ANTHROPIC_BASE_URL=https://your-gateway.example.com
export ANTHROPIC_AUTH_TOKEN=your-token
claude
```
