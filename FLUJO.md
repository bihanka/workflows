# Flujo Git Flow y GitHub Actions

Repositorio público: https://github.com/bihanka/workflows

El ejemplo del documento usa la cuenta `bsljvela`. Este proyecto está en la cuenta autenticada en esta computadora: `bihanka`.

## Ramas

| Rama | Quién la actualiza | Para qué sirve |
| --- | --- | --- |
| `main` | Solo por Pull Request | Código fuente estable. Está protegida. |
| `dev` | Desarrollo local | Integración de cambios antes de pasarlos a `main`. |
| `build` | GitHub Actions | Resultado de `npm run build` (carpeta `dist`) para el despliegue. |

## Parte 1. Repositorio remoto y primer commit

1. Se creó el repositorio público `workflows` con README.
2. En `C:\Users\lopez\ACTIONS` se inicializó Git, se creó `prueba1.txt` y se renombró la rama a `main`.
3. El remoto quedó en `https://github.com/bihanka/workflows.git`.
4. El primer push fue rechazado porque el remoto ya tenía el README.
5. `git pull origin main` falló con `refusing to merge unrelated histories`.
6. `git pull origin main --allow-unrelated-histories` unió el README remoto con el commit local.
7. `git push -u origin main` dejó `main` sincronizada.

Commit local: `Mi primer commit en main`.

## Parte 2. Rama dev

1. `git checkout -b dev` y `git push origin dev`.
2. En `dev` se modificó `README.md`, se eliminó `prueba1.txt` y se agregó `prueba2.txt`.
3. `git status` mostró las tres situaciones: modificado, eliminado y archivo nuevo.
4. Commit: `Commit desde la rama de Dev`.
5. Pull Request [#1](https://github.com/bihanka/workflows/pull/1) de `dev` hacia `main`, integrado con merge.

## Parte 3. Protección de main

Ruleset **Push validation**, activo, aplicado a `refs/heads/main`:

- Restrict deletions
- Require a pull request before merging (sin exigir revisores, para poder integrar el propio Pull Request)
- Block force pushes

Prueba de push directo:

```
git checkout main
git commit -m "Commit desde la rama main protegida"
git push origin main
```

GitHub respondió `GH013` y rechazó el push: *Changes must be made through a pull request*. Ese commit no está en el remoto. Quedó solo en la rama local `intento-main-protegida`.

A partir de ahí, `main` solo cambia con Pull Request.

## Parte 4. Proyecto Vite y GitHub Actions

El workflow ejecuta `npm install` y `npm run build`, así que el repositorio incluye un proyecto Vite en la raíz. Se generó con la plantilla vanilla (el equivalente no interactivo de `npm create vite@latest .`). `npm run build` escribe la carpeta `dist`.

El workflow está en `.github/workflows/deploy.yml`. Se dispara con un push a `main`. El job:

1. Hace checkout del repositorio.
2. Instala Node.js 22.
3. Ejecuta `npm install` y `npm run build`.
4. Publica el contenido de `dist` en la rama `build` con `s0/git-publish-subdir-action`.

Esos cambios entraron por el Pull Request [#2](https://github.com/bihanka/workflows/pull/2). Al hacer merge, Actions corrió solo y terminó en éxito:

https://github.com/bihanka/workflows/actions/runs/38198648641

La rama `build` quedó con `index.html`, `favicon.svg`, `icons.svg` y `assets/`.

Dos ajustes respecto al YAML del documento, necesarios para que el job pudiera completar:

- `permissions: contents: write`, porque el token automático de Actions solo lee el repositorio y sin permiso de escritura no puede crear `build`.
- `actions/setup-node` con Node.js 22, porque Vite 8 necesita una versión actual de Node.

El aviso *Failed to fetch target branch, probably doesn't exist* del primer run es esperado: `build` todavía no existía y la acción la creó.

## Evidencias

Capturas en la carpeta `evidencias/`:

- `01-ramas.png`: ramas `main`, `dev` y `build`
- `02-commits-main.png`: historial de commits en `main`
- `03-pull-requests.png`: los dos Pull Requests integrados
- `04-pr-dev.png` y `05-pr-workflow.png`: detalle de cada Pull Request
- `06-actions.png` y `07-actions-job.png`: ejecución exitosa del workflow
- `08-rama-build.png`: archivos publicados en `build`
- `09-rules.png`: ruleset Push validation sobre `main`

## Cómo seguir trabajando

Los cambios nuevos se hacen en `dev` (o en otra rama), se suben con push y se integran a `main` con un Pull Request. Cada merge a `main` vuelve a ejecutar el workflow y actualiza `build`.
