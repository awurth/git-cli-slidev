# Git CLI — Présentation Slidev

Talk de ~30 min en français sur les commandes et options git méconnues mais très utiles au quotidien.

## Commandes

```bash
pnpm run dev     # Dev server → http://localhost:3030
pnpm run build   # Build SPA statique
pnpm run export  # Export PDF (nécessite playwright-chromium)
```

## Stack

- Slidev `@slidev/cli ^52.16.0`
- Thème : `default`
- Langue : français
- Comark activé (`comark: true`)

## Sujets couverts

1. **Staging sélectif** — `git add -p`, `restore -p`, `stash -p`, `add -N`
2. **Diff lisible** — `diff -w`, `--color-moved`, delta, diff-so-fancy
3. **Navigation** — `switch`/`restore` vs checkout, `-` branche précédente, `branch -v`
4. **Historique** — rebase interactif, `abbreviateCommands`, `instructionFormat`, fixup/autosquash, `git history` (expérimental)
5. **Parallélisme** — worktrees *(section désactivée)*
6. **Sécurité** — `push --force-with-lease`, rerere/rerere.autoupdate
7. **Config** — gitignore global
8. **Aliases** — réduire la frappe au maximum

## Conventions

- Slides séparés par `---`
- Layout de section : `layout: section`
- Comparaisons : `layout: two-cols` + `::right::`
- Révélations progressives : `v-click` / `<v-clicks>`
- Highlighting de code progressif : ` ```bash {1|2|3} `
- Notes présentateur : `<!-- notes ici -->`

## Fichiers

- `slides.md` — contenu principal
- `example.md` — référence Slidev (ne pas modifier)
