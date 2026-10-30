---
description: Instala una skill y sus comandos complementarios en .opencode
---

# Instalar skill en este proyecto

Argumentos recibidos: `$ARGUMENTS`

## Regla obligatoria

Todas las skills de este proyecto viven en `.opencode/skills/<nombre>/SKILL.md`.
Nunca dejarlas en `.agents/`, `.claude/` ni en otro directorio.

## Proceso

1. Si `$ARGUMENTS` está vacío, pide el paquete o URL a instalar (ejemplo: `vercel-labs/skills@find-skills`) y detente.

2. Normaliza `$ARGUMENTS`: elimina prefijos como `npx skills add`, `npx --yes skills add`, la opción `--skill`, comillas y barras finales. Después identifica la fuente y el nombre de la skill:
   - URL de GitHub con `/blob/.../SKILL.md`: repo = `https://github.com/<owner>/<repo>`, skill = carpeta padre del `SKILL.md`.
   - `owner/repo@skill`: repo = `https://github.com/owner/repo`, skill = `skill`.
   - Otro paquete: instálalo con `npx --yes skills add <paquete> --agent opencode -y`, mueve `.agents/skills/*` a `.opencode/skills/`, elimina `.agents` y salta al paso 7.

3. Clona el repo en un directorio temporal:

   ```bash
   tmp=$(mktemp -d)
   git clone --depth 1 <repo> "$tmp"
   ```

4. Copia solo la carpeta de la skill a `.opencode/skills/<skill>`:
   - Si ya existe, no la sobrescribas sin avisar al usuario.

5. Copia los comandos complementarios si existen:
   - Busca en `$tmp/.opencode/commands`, `$tmp/.opencode/command` y `$tmp/commands`.
   - Copia a `.opencode/commands/` solo los archivos `.md` o `.toml` cuyo nombre empiece por `<skill>` (ejemplo: `ponytail-review.md`).
   - No sobrescribas archivos existentes sin avisar.

6. Plugins: solo cópialos si el README o INSTALL del repo dice que son necesarios y no importan archivos fuera de su carpeta. Si dependen de `hooks/` u otros directorios, NO los copies: avisa y usa el mecanismo oficial del repo. Para ponytail, el plugin no es necesario porque `AGENTS.md` ya aplica las reglas.

7. Verifica que exista `.opencode/skills/<skill>/SKILL.md` y lista los comandos copiados.

8. Limpia el temporal: `rm -rf "$tmp"`.

9. Responde con las rutas finales y un resumen de 1-3 líneas. No instales nada de forma global ni en `.claude/`.
