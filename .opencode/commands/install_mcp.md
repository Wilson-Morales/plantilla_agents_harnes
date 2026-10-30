---
description: Instala un servidor MCP local o remoto en opencode.json
---

# Instalar MCP en este proyecto

Argumentos recibidos: `$ARGUMENTS`

## Formato

- Local: `/install_mcp <nombre> local <comando...>`
  Ejemplo: `/install_mcp context7 local npx -y @upstash/context7-mcp`
- Remoto: `/install_mcp <nombre> remote <url>`
  Ejemplo: `/install_mcp sentry remote https://mcp.sentry.dev/mcp`
- Instalador del propio MCP: `/install_mcp npx <cli> setup`
  Ejemplo: `/install_mcp npx ctx7 setup` → deduce el MCP (`context7`) y añade la entrada manualmente.
- JSON de Claude/otros:

  ```json
  {
    "mcpServers": {
      "chrome-devtools": {
        "command": "npx",
        "args": ["-y", "chrome-devtools-mcp@latest"]
      }
    }
  }
  ```

Opciones: `--env KEY=VALUE`, `--header KEY=VALUE`, `--cwd <ruta>`, `--timeout <ms>`, `--oauth`, `--no-oauth`.

Si faltan el nombre o el comando/URL, pídelos y detente.

## Conversión de JSON `mcpServers`

- `command` + `args` → `type: "local"` y `command: [command, ...args]`.
- `env` → `environment`.
- `url` → `type: "remote"` y `url`.
- `headers` se conserva.
- `type: "stdio"` → `"local"`; `"sse"`, `"http"` o `"streamable-http"` → `"remote"`.

## Reglas

- Configura solo el proyecto: `opencode.json` o `opencode.jsonc` en la raíz.
- No modifiques `~/.config/opencode/opencode.jsonc` salvo que el usuario lo pida explícitamente.
- No uses `opencode mcp add`: escribe en la configuración global por defecto.
- No ejecutes instaladores como `ctx7 setup`: escriben en la config global y en `.agents/skills`. Deduce el MCP y escribe la entrada en `opencode.json`.
- Nunca escribas secretos en texto plano; usa `{env:VARIABLE}`.
- Si `mcp.<nombre>` ya existe, no lo sobrescribas sin avisar.

## Proceso

1. Busca `opencode.json` o `opencode.jsonc` en la raíz. Si no existe, crea `opencode.json`:

   ```json
   {
     "$schema": "https://opencode.ai/config.json",
     "mcp": {}
   }
   ```

2. Añade la entrada dentro de `mcp`:

   Local:

   ```json
   "<nombre>": {
     "type": "local",
     "command": ["<comando>", "<arg>"],
     "enabled": true
   }
   ```

   Remoto:

   ```json
   "<nombre>": {
     "type": "remote",
     "url": "<url>",
     "enabled": true
   }
   ```

3. Añade solo las opciones pedidas: `environment`, `cwd`, `headers`, `oauth` o `timeout`.

4. Valida que el archivo siga siendo JSON/JSONC válido.

5. Recarga y verifica:

   ```bash
   opencode reload
   opencode mcp list --print-logs
   ```

   Un servidor local puede aparecer `pending` unos segundos; vuelve a listar hasta ver `connected`. Para servidores remotos con autenticación:

   ```bash
   opencode mcp debug <nombre>
   opencode mcp auth <nombre>
   ```

6. Responde con el archivo modificado, el nombre del MCP y recuerda reiniciar opencode si no queda conectado.
