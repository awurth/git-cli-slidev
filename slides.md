---
theme: default
background: https://cover.sli.dev
title: 'Apprendre à aimer Git en ligne de commande'
info: false
class: text-center
transition: slide-left
comark: true
duration: 30min
---

# Apprendre à aimer Git en ligne de commande

Les commandes, options et réglages méconnues mais très utiles au quotidien

<div class="pt-8 text-gray-400">
  Alexis Wurth · SensioLabs
</div>

<!--
Intro : qui utilise git en CLI ? Qui utilise uniquement l'interface graphique ?
L'objectif aujourd'hui c'est de vous donner des outils concrets pour aller plus vite.
-->

---
---

<div class="flex items-center gap-8 h-full">
  <img src="/avatar.jpg" alt="Alexis Wurth" class="w-36 h-36 rounded-full object-cover ring-4 ring-gray-200 shadow-lg shrink-0" />
  <div>
    <h1>Alexis Wurth</h1>
    <p>Développeur PHP/Symfony</p>
    <p class="flex items-center gap-2"><img src="./github.svg" class="w-5 h-5" alt="GitHub" /><a href="https://github.com/awurth">awurth</a></p>
  </div>
</div>

<img src="./sensiolabs.png" class="absolute bottom-10 right-12 h-14" alt="SensioLabs" />
<img src="./qr-slides.png" class="absolute bottom-10 left-12 h-14" alt="Slides" />

---
layout: default
---

# Au programme

<v-clicks>

- **Regarder ses changements** — diff, delta, color-moved, log
- **Naviguer** — switch/restore, branch -v
- **Mettre de côté** — stash
- **Préparer le commit** — add -N, add -p, commit.verbose
- **Rebase** — rebase -i, autoStash, update-refs
- **Pousser** — pull.rebase, force-with-lease
- **Conflits** — zdiff3, rerere
- **Config & aliases** — gitignore global, includeIf, aliases

</v-clicks>

---
layout: section
---

# Regarder ses changements

Comprendre ce qui change, ce qui a changé

---
layout: default
---

# `status.showUntrackedFiles`

Par défaut, git regroupe les fichiers non-trackés sous leur dossier parent.

```bash
$ git status
Untracked files:
  src/   # ← on ne voit pas ce qu'il y a dedans
```

<v-click>

```bash
git config --global status.showUntrackedFiles all
```

```bash
$ git status
Untracked files:
  src/Controller/UserController.php
  src/Repository/UserRepository.php
  src/Entity/User.php
```

> Voir tous les fichiers, pas juste le dossier — utile avant un `git add`.

</v-click>

<v-click>

Ou ponctuellement :

```bash
git status -u
```

</v-click>

<!--
Comportement par défaut frustrant quand on découvre un repo ou un nouveau dossier.
Avec `all`, on voit exactement ce qui sera stagé.
-->

---
layout: default
---

# `git add -N` — intention d'ajouter

Nouveaux fichiers **invisibles** à `git diff` et `git add -p` par défaut.

```bash
# Nouveau fichier : git diff ne le montre pas
touch src/new-feature.php
git diff  # → rien

# Avec -N (--intent-to-add)
git add -N src/new-feature.php
git diff  # → montre le fichier entier comme ajout
```

<v-click>

**Utilité** : visualiser l'ensemble de ses modifications d'un coup, nouveaux fichiers inclus, avant de committer.

```bash
# Workflow courant
git add -N .          # marquer tous les nouveaux fichiers
git diff              # voir tout ce qui change
git add -p            # indexer sélectivement
```

</v-click>

<v-click>

💡 **Alias utile** : `git ap = !git add -N . && git add -p`

</v-click>

<!--
Petit tip mais qui fait gagner du temps quand on a créé plusieurs nouveaux fichiers.
L'alias ap combine les deux étapes du workflow courant en une seule commande.
-->

---
layout: two-cols
---

# delta

Remplacer le pager par défaut (`less`) par **delta**.

```bash
# Installation
brew install git-delta

# Configuration
git config --global core.pager delta
git config --global interactive.diffFilter "delta --color-only"
```

**Fonctionnalités :**
- Syntax highlighting dans les diffs
- Affichage côte-à-côte (`--side-by-side`)
- Numéros de lignes
- Thèmes personnalisables

::right::

<v-click>

# diff-so-fancy

Alternative plus légère.

```bash
npm install -g diff-so-fancy

git config --global core.pager \
  "diff-so-fancy | less --tabs=4 -RFX"
```

**Différences avec delta :**
- Plus simple à configurer
- Moins de fonctionnalités
- Rendu épuré

> Les deux transforment radicalement la lisibilité des diffs. À tester absolument.

</v-click>

<!--
Montrer une capture d'écran ou une démo si possible. L'impact visuel est immédiat.
delta : https://github.com/dandavison/delta
diff-so-fancy : https://github.com/so-fancy/diff-so-fancy

En préparant ces slides j'ai découvert un concurrent : hunk (https://github.com/modem-dev/hunk) qui se prétend meilleur que les deux. Pas testé.
-->

---
layout: default
---

# `--color-moved`

Colorier les blocs déplacés en couleur distincte — parfait pour les refactorings.

```bash
git diff --color-moved           # blocs déplacés en couleur distincte
git diff -w --color-moved        # ignorer les espaces en plus
```

<v-click>

**Activer par défaut :**

```bash
git config --global diff.colorMoved default
git config --global diff.colorMovedWS allow-indentation-change
```

On voit immédiatement ce qui bouge vs ce qui change vraiment.

</v-click>

<!--
color-moved : disponible depuis git 2.15 (2017). Beaucoup de devs ne savent pas que ça existe.
diff.colorMoved peut valoir : no, default, blocks, zebra, dimmed-zebra
-w / --ignore-all-space : utile ponctuellement pour ignorer le bruit d'indentation.
-->

---
layout: default
---

# `git log` — options essentielles

```bash
# Compact + graphe de branches
git log --oneline --graph --decorate

# Limiter / filtrer
git log -10
git log --since="2 weeks ago"
git log --author="Alexis"
git log -- src/Controller/
```

<v-click>

```bash
# Voir les modifications
git log -p          # avec le diff complet
git log --stat      # fichiers modifiés + stats

# Chercher dans les messages
git log --grep="fix"

# Chercher dans le contenu des commits
git log -S "password"        # commits où "password" apparaît/disparaît
git log -G "password.*hash"  # commits dont le diff matche la regex
```

</v-click>

<!--
--stat : vue rapide de l'ampleur d'un commit. Très utile en code review.
--format : pratique pour générer des changelogs ou rapports.
-S (pickaxe) : cherche les commits qui ont ajouté ou supprimé la chaîne exacte — idéal pour retrouver quand une fonction a été introduite ou supprimée.
-G : comme -S mais avec une regex sur le contenu du diff — plus puissant, légèrement plus lent.
Les deux peuvent être lents sur un gros dépôt : limiter avec --since, -n ou -- path/ pour réduire le scope.
-L :début,fin:fichier — suit l'historique d'une plage de lignes ou d'une fonction précise. Ex: git log -L :maFonction:src/Foo.php
--follow : suit l'historique d'un fichier à travers les renommages. Ex: git log --follow src/Foo.php
-->

---
layout: section
---

# Naviguer

Se déplacer dans le dépôt

---
layout: two-cols-header
---

# `switch` et `restore` — checkout fait trop de choses

::left::

**Avant (git checkout)**

```bash
# Changer de branche
git checkout main

# Créer et changer de branche
git checkout -b feature/foo

# Restaurer un fichier
git checkout -- src/foo.php

# Restaurer depuis un commit
git checkout abc123 -- src/foo.php
```

😵 Une seule commande, quatre comportements différents

::right::

<v-click>

**Maintenant (git 2.23+, 2019)**

```bash
# Changer de branche
git switch main

# Créer et changer de branche
git switch -c feature/foo

# Restaurer un fichier (working tree)
git restore src/foo.php

# Restaurer depuis un commit
git restore --source=abc123 src/foo.php

# Désindexer
git restore --staged src/foo.php
```

✅ Intentions claires, moins d'erreurs

</v-click>

<!--
checkout reste disponible et fonctionnel. switch/restore sont juste plus explicites.
Disponible depuis git 2.23 (août 2019).
-->

---
layout: default
---

# Le tiret `-` : branche précédente

Fonctionne comme `cd -` dans le shell.

```bash
git switch feature/foo
# ... travail ...
git switch main
git switch -          # → retour sur feature/foo
```

```bash
# Fonctionne aussi avec les anciennes commandes
git checkout -
git merge -           # merger la branche précédente
git rebase -          # rebaser sur la branche précédente
```

<!--
Cas concret : hotfix rapide
git switch -c hotfix/urgent
... correction ...
git switch main
git merge -           # merger hotfix depuis main
git switch -          # retour sur sa branche de feature
-->

<!--
Simple mais méconnu. Économise beaucoup de temps sur les allers-retours entre deux branches.
-->

---
layout: default
---

# `git branch -v`

Voir l'état de ses branches en un coup d'œil.

```bash
git branch -v
```

```
  feature/auth     a3f2e1c Add JWT middleware
  feature/search   8b4d2f0 WIP: elastic integration
* main             c1e9a4b Merge pull request #42
  hotfix/login     f2a1c3d Fix session timeout
  old-feature      f2a1c3d [gone] WIP
```

<v-clicks>

```bash
# -vv : voir aussi le tracking remote
git branch -vv
```

```
  feature/auth  a3f2e1c [origin/feature/auth] Add JWT middleware
* main          c1e9a4b [origin/main: behind 3] Merge pull request #42
  old-feature   f2a1c3d [origin/old-feature: gone] WIP
```

```bash
# Trier par date de dernier commit
git branch --sort=-committerdate -v

# Activer par défaut
git config --global branch.sort -committerdate
```

</v-clicks>

<!--
branch -vv révèle aussi les branches qui ont divergé du remote. Très utile en équipe.
-->

---
layout: section
---

# Mettre de côté

Changer de contexte sans perdre son travail

---
layout: default
---

# `git stash push` — options utiles

```bash
# Inclure les fichiers non-trackés
git stash push -u

# Nommer son stash
git stash push -m "wip: refacto contrôleur user"

# Tout combiner
git stash push -u -m "wip: nouvelle feature"
```

<v-click>

```bash
# git stash list — les noms rendent la liste lisible
stash@{0}: On main: wip: refacto contrôleur user
stash@{1}: On main: wip: nouvelle feature
# Sans -m → stash@{2}: WIP on main: abc1234 fix login
```

</v-click>

<!--
-u : évite la surprise de perdre des fichiers non-trackés quand on change de branche
Nommer ses stashs = indispensable dès qu'on en accumule plusieurs
-->

---
layout: default
---

# `git stash pop` — restaurer l'état exact

```bash
# Restaurer aussi l'état staged/unstaged
git stash pop --index

# Ou en config pour ne jamais y penser
git config --global stash.index true
```

> `stash.index true` = `--index` automatique à chaque pop/apply.

<v-click>

```bash
# Alias — ne plus jamais oublier -u et -m
git config --global alias.sw '!git stash push -u -m'
# Usage : git sw "wip: ma feature"
```

</v-click>

<!--
stash.index = true évite de devoir taper --index à chaque pop/apply : restaure l'état staged/unstaged automatiquement.
L'alias sw (stash work) rend le flux naturel : git sw "contexte", git stash pop.
-->

---
layout: section
---

# Préparer un commit

Choisir exactement ce qui part dans le commit

---
layout: default
---

# `git add -p`

Le **mode patch** : sélectionner les hunks (gros morceaux) à indexer un par un — bugfix, refactoring, debug, sans toucher aux fichiers.

```bash
git add -p
```

<v-click>

| Touche | Action |
|--------|--------|
| `y` | Indexer ce hunk |
| `n` | Ignorer ce hunk |
| `s` | Découper en plus petits hunks |
| `e` | Éditer manuellement |
| `q` | Quitter |

> 💡 `-p` fonctionne aussi avec `restore`, `stash`, `reset`, `commit`

</v-click>

<!--
Démonstration live si possible. C'est souvent une révélation pour les devs qui ne connaissent pas.
La symétrie est intentionnelle : -p fait toujours la même chose, quelle que soit la commande.
-->

---
layout: default
---

# `commit.verbose` — voir le diff dans l'éditeur

Par défaut, l'éditeur de commit ne montre rien du tout.

```bash
# Ponctuellement
git commit -v
git commit --verbose
```

<v-click>

```bash
# Toujours actif
git config --global commit.verbose true
```

L'éditeur affiche le diff complet sous le message — plus besoin d'un terminal séparé pour se rappeler ce qu'on committe.

</v-click>

<!--
Particulièrement utile pour les commits multi-fichiers : on écrit un message précis en voyant exactement ce qui part.
-->

---
layout: section
---

# Rebase

Réécrire l'historique avant de pousser

---
layout: default
---

# Rebase interactif

Réécrire les N derniers commits avant de pousser.

```bash
git rebase -i HEAD~5
```

<v-click>

```
pick a1b2c3d Ajouter le formulaire de connexion
pick b2c3d4e WIP
pick c3d4e5f fix typo
pick d4e5f6g Ajouter les tests
```

</v-click>

<v-click>

| Commande | Action |
|----------|--------|
| `pick` / `p` | Garder le commit tel quel |
| `reword` / `r` | Modifier le message |
| `squash` / `s` | Fusionner avec le précédent (garder msg) |
| `fixup` / `f` | Fusionner avec le précédent (supprimer msg) |
| `drop` / `d` | Supprimer le commit |

</v-click>

<!--
Règle d'or : ne jamais rebase ce qui est déjà sur le remote partagé.
-->

---
layout: default
---

# Améliorer le rendu du rebase

Par défaut, les commandes sont en toutes lettres. On peut faire mieux.

```bash
# Abréger les commandes (pick → p, squash → s, etc.)
git config --global rebase.abbreviateCommands true

# Afficher l'auteur dans la liste
# https://git-scm.com/docs/git-log#Documentation/git-log.txt-formatformat-string
git config --global rebase.instructionFormat "[%an @ %ar] %s"
```

<v-click>

**Avant :**
```
pick a1b2c3d Ajouter le formulaire de connexion
pick b2c3d4e WIP
squash c3d4e5f fix typo
```

**Après :**
```
p a1b2c3d [Alice @ 2 days ago] Ajouter le formulaire de connexion
p b2c3d4e [Bob @ 5 hours ago] WIP
s c3d4e5f [Alice @ 3 hours ago] fix typo
```

</v-click>

<!--
Sur une branche de feature à plusieurs, l'auteur évite les mauvaises surprises.
-->

---

# `rebase.autoStash`

Lancer un rebase avec des modifications non commitées.

```bash
git config --global rebase.autoStash true
```

<v-click>

Sans autostash :
```
error: cannot rebase: You have unstaged changes.
```

</v-click>

<v-click>

Avec autostash, git fait automatiquement :
```bash
git stash        # avant le rebase
# ... rebase ...
git stash pop    # après le rebase
```

Fini les `git stash` manuels avant chaque rebase.

</v-click>

---
layout: two-cols
---

# `rebase --update-refs`

Branches empilées, `main` avance :

```
main ── A ── A'
         └── feat-a ── B ── C
                              └── feat-b ── D ── E
```

Sans l'option, il faut rebaser chaque branche à la main, dans l'ordre :

```bash
git rebase main feat-a
git rebase feat-a feat-b
```

::right::

<v-click>

**Avec `--update-refs`** : une seule commande depuis le sommet.

```bash
git switch feat-b
git rebase -i --update-refs main
```

Lignes `u` insérées automatiquement :

```{2}
pick b3f1a2c Commit sur feat-a
u refs/heads/feat-a
pick d9e4b1f Commit sur feat-b
pick a7c2d0e Autre commit feat-b
```

</v-click>

<v-click>

**Activer par défaut :**
```bash
git config --global rebase.updateRefs true
```

⚠️ Peut capturer des branches inattendues — vérifier les lignes `u` dans l'éditeur.

</v-click>

<!--
Git 2.38+. Idéal pour les stacked PRs.
-->

---
layout: default
---

# `git history` — commandes expérimentales

<div class="text-orange-400 font-bold mb-4">⚠️ Expérimental — git 2.54+ requis (2026)</div>

Deux nouvelles commandes de haut niveau pour réécrire l'historique.

```bash
# Renommer un commit sans passer par rebase -i
git history reword a1b2c3d
# → ouvre l'éditeur directement sur le message du commit
```

```bash
# Découper un gros commit en plusieurs commits
git history split a1b2c3d
# → replace HEAD sur ce commit, vous permet de re-committer morceau par morceau
```

<v-click>

> Ces commandes sont encore en développement. L'API peut changer.

</v-click>

<!--
reword évite d'ouvrir toute la liste des commits juste pour modifier un message — pas besoin d'ouvrir toute la liste.
Introduites dans git 2.54 (2026). Vérifier disponibilité : git history --help
-->

---
layout: section
---

# Pousser

Envoyer son travail sur le remote

---
layout: default
---

# `pull.rebase` — éviter les merge commits parasites

```bash
git pull  # sans config = merge commit si divergence
# → Merge branch 'main' of github.com/foo/bar  ← bruit dans l'historique
```

<v-click>

```bash
git config --global pull.rebase true
# git pull = git fetch + git rebase
```

Historique linéaire, pas de merge commits inutiles.

</v-click>

<v-click>

```bash
# Variante : préserver les merges locaux intentionnels
git config --global pull.rebase merges
```

> `true` pour la plupart des cas. `merges` si votre branche contient des merges intentionnels à conserver.

</v-click>

<!--
pull.rebase true : équivalent à toujours passer --rebase à git pull.
Combine bien avec rebase.autoStash true pour ne pas bloquer sur des modifs non commitées.
-->

---

# `push.autoSetupRemote`

Fini le `git push -u origin HEAD` sur chaque nouvelle branche.

<v-click>

```bash
# Sans le réglage
git switch -c ma-feature
git push   # ✗ fatal: The current branch ma-feature has no upstream branch.
           #   To push the current branch and set the remote as upstream, use
           #   git push --set-upstream origin ma-feature
```

</v-click>

<v-click>

```bash
git config --global push.autoSetupRemote true

git switch -c ma-feature
git push   # ✓ push + tracking configuré automatiquement
```

</v-click>

<!--
Disponible depuis git 2.37 (2022).
Équivalent à toujours passer --set-upstream, mais sans y penser.
Compatible avec push.default = simple (défaut depuis git 2.0).
-->

---
layout: default
---

# `push --force-with-lease`

`git push --force` devrait presque toujours être remplacé.

<v-click>

**Le problème :**
```bash
# Bob a poussé un commit pendant qu'Alice rebasait
git push --force   # ← Alice écrase le commit de Bob silencieusement
```

**La solution :**
```bash
git push --force-with-lease --force-if-includes
# Si quelqu'un a poussé entre-temps → REJET avec erreur claire

# Activer --force-if-includes automatiquement (git 2.30+, 2020)
git config --global push.useForceIfIncludes true

# Alias indispensable
git config --global alias.pf "push --force-with-lease"
git pf
```

</v-click>

<!--
force-with-lease existe depuis git 1.8.5 (2013). Pas d'excuse pour utiliser --force.
force-if-includes : protection supplémentaire si le fetch a été fait mais pas intégré.
push.useForceIfIncludes : active --force-if-includes automatiquement sur tous les push --force-with-lease.
-->

---
layout: section
---

# Gérer les conflits

Résoudre intelligemment

---
layout: two-cols-header
---

# `merge.conflictStyle zdiff3`

Mieux comprendre les conflits grâce au contexte ancêtre.

::left::

**Style par défaut (`merge`) :**

```
<<<<<<< HEAD
return $user->getEmail();
=======
return $user->getUsername();
>>>>>>> feature/login
```

😕 Impossible de savoir pourquoi ça a divergé.

::right::

<v-click>

**Avec `zdiff3` :**

```
<<<<<<< HEAD
return $user->getEmail();
||||||| base
return $user->getName();
=======
return $user->getUsername();
>>>>>>> feature/login
```

✅ Le bloc `|||||||` montre le code **avant** les deux modifications.

```bash
git config --global merge.conflictStyle zdiff3
```

Disponible depuis git 2.35 (2022). `zdiff3` améliore `diff3` en réduisant les faux conflits.

</v-click>

<!--
zdiff3 = "zealous diff3". Moins de conflits parasites que diff3 classique.
Indispensable avec rerere : la résolution est plus intelligente avec un meilleur contexte ancêtre.
-->

---
layout: default
---

# rerere — Reuse Recorded Resolution

Git mémorise les résolutions de conflits pour les rejouer automatiquement.

```bash
# Activer rerere
git config --global rerere.enabled true

# Mettre à jour l'index automatiquement après résolution
git config --global rerere.autoUpdate true
```

<v-click>

**Cas d'usage typique :**
```bash
# Branche longue qu'on rebase régulièrement sur main
git rebase main    # → conflit sur src/Service/Auth.php
# Résoudre le conflit manuellement → rerere enregistre

# La semaine suivante, même rebase
git rebase main    # → rerere rejoue automatiquement ✅
```

```bash
git rerere status  # voir les résolutions enregistrées
git rerere diff    # voir ce que rerere va appliquer
```

</v-click>

<!--
rerere.autoUpdate : applique la résolution ET indexe le fichier. Sans ça, il faut faire git add manuellement.
Très utile sur les projets où une branche de feature vit longtemps en parallèle de main.

Si vous avez souvent des conflits difficiles à gérer lors des rebases, regardez aussi du côté de `git rebase --onto` : ça permet de rebaser uniquement une partie d'une branche, et d'éviter de rejouer des conflits liés à des commits qui n'ont rien à voir avec ce qu'on veut intégrer.
-->

---
layout: section
---

# Configurer son environnement

Mettre en place une fois, en profiter partout

<!--
Bonus : help.autocorrect
git config --global help.autocorrect immediate
→ git statsu : exécute 'status' directement, sans délai ni confirmation.
(valeur numérique = délai en dixièmes de seconde avant exécution)
-->

---
layout: default
---

# `.gitignore` global

Ne pas polluer le `.gitignore` du projet avec ses outils personnels.

```bash
# Configurer un gitignore global
git config --global core.excludesFile ~/.gitignore_global
```

<v-click>

**Contenu typique de `~/.gitignore_global` :**

```
.DS_Store        # macOS
.idea/           # JetBrains
.vscode/         # VS Code
*.swp            # Vim
*.local
.env.local
```

> Si c'est lié à **votre machine ou votre outil** → global. Si c'est lié au **projet** → `.gitignore` du projet.

</v-click>

<!--
Évite les PR polluées par des fichiers .DS_Store ou .idea.
À faire une fois et ça s'applique à tous vos projets.
-->

---
layout: default
---

# `includeIf` — config par contexte

Adapter la config git selon le projet, sans tout mélanger.

```ini
# ~/.gitconfig
[user]
  name = Alexis Wurth
  email = alexis@perso.dev

[includeIf "gitdir:~/Projects/sensiolabs/"]
  path = ~/.gitconfig-work
```

<v-click>

```ini
# ~/.gitconfig-work
[user]
  email = alexis.wurth@sensiolabs.com
  signingKey = ABCD1234

[commit]
  gpgSign = true
```

**Autres conditions :** `gitdir:`, `onbranch:release/**`, `hasconfig:remote.*.url:`

> Une seule `~/.gitconfig`, comportements différents par projet.

</v-click>

<!--
Disponible depuis git 2.13 (2017). Très utile pour séparer config perso et pro.
Le trailing slash dans gitdir: est important : ~/work/ matche tout ce qui est dans ce dossier.
-->

---
layout: default
---

# `fetch.prune` — Rester en sync avec le remote

Les branches supprimées sur le remote restent visibles localement par défaut.

```bash
git branch -vv
#   old-feature  f2a1c3d [origin/old-feature: gone] WIP  ← branche fantôme
```

<v-click>

```bash
# Nettoyer manuellement
git fetch --prune

# Activer en config
git config --global fetch.prune true
git config --global fetch.pruneTags true
```

> `fetch.prune true` = le remote fait loi. Fini les branches fantômes.

</v-click>

<v-click>

```bash
# Supprimer toutes les branches locales dont le remote est gone
git branch -vv | awk '/: gone]/{print $1}' | xargs git branch -D
```

</v-click>

<!--
Sans fetch.prune, git branch -r affiche des branches supprimées indéfiniment.
fetch.pruneTags : disponible depuis git 2.17 (2018).
-D : force la suppression même si non mergée.
-->

---
layout: default
---

# Pourquoi des aliases ?

`git status` → **11 caractères**, 50 fois par jour = **550 caractères** pour rien

`git s` → **5 caractères** ✅

<v-click>

**Objectif :** max 3-4 caractères après `git` pour les commandes fréquentes.

```bash
# Voir ses aliases actuels
git config --list | grep alias

# Savoir d'où vient chaque config
git config --list --show-origin
```

> Un alias bien choisi doit se retenir en 10 secondes. S'il faut l'expliquer, il est trop cryptique.

</v-click>

---
layout: default
---

# Aliases git essentiels

```ini {all|1-5|6-11|12-16}
[alias]
  # Navigation
  s    = status -s
  br   = branch -vv --sort=-committerdate
  sw   = switch

  # Diff
  d    = diff
  dw   = diff -w
  dc   = diff --cached
  ds   = diff --stat

  # Commit
  ap   = add -p
  cm   = commit -m
  fix  = commit --fixup
  rb   = rebase -i
  pf   = push --force-with-lease

  # Log
  l    = log --oneline --graph --decorate -20
  la   = log --oneline --graph --decorate --all
```

<!--
Adapter selon ses habitudes. L'essentiel : commencer par les commandes qu'on tape le plus souvent.
-->

---
layout: default
---

# Shell aliases — aller encore plus loin

Réduire `git` lui-même.

```bash
# Dans ~/.zshrc ou ~/.bashrc

# Alias principal
alias g='git'

# Raccourcis directs
alias gs='git status -s'
alias gl='git log --oneline --graph --decorate -20'
alias gd='git diff'
alias gdc='git diff --cached'
alias gco='git checkout'
alias gsw='git switch'
alias gp='git push'
alias gpf='git push --force-with-lease'
alias grb='git rebase -i'
```

<v-click>

```bash
# Résultat
g s            # git status -s
g l            # git log --oneline...
g sw feature   # git switch feature
g pf           # git push --force-with-lease
```

</v-click>

<!--
Oh My Zsh/Fish incluent déjà des alias git. Vérifier avant de dupliquer.
Oh My Zsh git plugin : https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git
gst = git status, gco = git checkout, etc.
-->

---

# À découvrir aussi

<v-clicks>

- `git config --global diff.algorithm histogram` — algorithme de diff plus lisible sur certains fichiers
- `git commit --fixup` — créer automatiquement un commit de correction pour l'autosquash
- `git bisect` — trouver le commit qui a introduit un bug par dichotomie
- `git worktree` — plusieurs branches en parallèle dans des dossiers séparés
- `git rebase --onto` — transplanter des commits d'une base vers une autre (branche depuis la mauvaise base, supprimer des commits au milieu)

</v-clicks>

---
layout: center
class: text-center
---

# En résumé

**Choisissez 2-3 choses à adopter aujourd'hui :**

<v-clicks>

- `git add -p` — commits plus propres
- `push --force-with-lease` — ne plus écraser le travail d'un collègue
- `rerere.enabled` — si vous rebasez souvent
- delta — diffs agréables à lire
- 3-4 aliases — les commandes les plus fréquentes

</v-clicks>

<!--
Questions ?

Liens :
- delta : https://github.com/dandavison/delta
- diff-so-fancy : https://github.com/so-fancy/diff-so-fancy
- git documentation : https://git-scm.com/docs
-->
