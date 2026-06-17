---
theme: default
background: https://cover.sli.dev
title: 'Git en ligne de commande : ce qu''on ne vous a pas appris'
info: false
class: text-center
transition: slide-left
comark: true
duration: 30min
---

# Git en ligne de commande

## Ce qu'on ne vous a pas appris

Les commandes et options méconnues mais très utiles au quotidien

<div class="pt-8 text-gray-400">
  Alexis Wurth · Sensiolabs
</div>

<!--
Intro : qui utilise git en CLI ? Qui utilise uniquement l'interface graphique ?
L'objectif aujourd'hui c'est de vous donner des outils concrets pour aller plus vite.
-->

---
layout: default
---

# Au programme

<v-clicks>

- **Staging sélectif** — `add -p`, `restore -p`, `stash -p`, `add -N`
- **Diff plus lisible** — `-w`, `--color-moved`, delta, diff-so-fancy
- **Naviguer plus vite** — `switch`, `restore`, le tiret `-`, `branch -v`
- **Réécrire l'historique** — rebase interactif, fixup/autosquash, `git history`
- **Travailler en parallèle** — worktrees
- **Sécurité** — `push --force-with-lease`, rerere
- **Aliases** — taper le moins possible

</v-clicks>

<!--
On va couvrir beaucoup de terrain. Si vous ne retenez que 2-3 choses aujourd'hui, c'est déjà une victoire.
-->

---
layout: section
---

# Staging sélectif

Committer exactement ce qu'on veut

---
layout: two-cols
---

# `git add -p`

Le **mode patch** : sélectionner les hunks à indexer un par un.

```bash
git add -p
git add --patch
```

<v-clicks>

Réponses disponibles à chaque hunk :

| Touche | Action |
|--------|--------|
| `y` | Indexer ce hunk |
| `n` | Ignorer ce hunk |
| `s` | Découper en plus petits hunks |
| `e` | Éditer manuellement |
| `q` | Quitter |
| `?` | Aide |

</v-clicks>

::right::

<v-click>

**Pourquoi c'est précieux ?**

Un fichier modifié peut contenir :
- Un bugfix → à committer maintenant
- Un refactoring → à committer séparément
- Du debug temporaire → à ne PAS committer

`git add -p` permet de séparer tout ça **sans toucher aux fichiers**.

</v-click>

<!--
Démonstration live si possible. C'est souvent une révélation pour les devs qui ne connaissent pas.
-->

---
layout: default
---

# Le `-p` est partout

Le mode patch existe sur plusieurs commandes.

<v-clicks>

```bash
# Indexer partiellement
git add -p

# Désindexer partiellement (unstage)
git restore --staged -p

# Annuler des modifications partiellement (dans le working tree)
git restore -p

# Stasher partiellement
git stash -p
```

</v-clicks>

<v-click>

> La symétrie est intentionnelle : `-p` fait toujours la même chose, quelle que soit la commande.

</v-click>

<!--
restore -p sans --staged : annule dans le working tree (attention, irréversible si pas indexé)
-->

---
layout: default
---

# `git add -N` — intention d'ajouter

Nouveaux fichiers **invisibles** à `git diff` par défaut.

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

<!--
Petit tip mais qui fait gagner du temps quand on a créé plusieurs nouveaux fichiers.
-->

---
layout: section
---

# Diff plus lisible

Comprendre ce qui change vraiment

---
layout: two-cols
---

# `git diff -w` et `--color-moved`

**Ignorer les espaces :**

```bash
git diff -w
# équivalent à :
git diff --ignore-all-space
```

Utile après un reformatage ou une correction d'indentation.

<v-click>

```bash
# Variantes plus fines
git diff --ignore-space-change   # -b
git diff --ignore-blank-lines    # -B
```

</v-click>

::right::

<v-click>

**Colorier les blocs déplacés :**

```bash
git diff --color-moved
git diff --color-moved=blocks
```

Un bloc de code **déplacé** s'affiche en couleur distincte au lieu d'apparaître comme suppression + ajout.

</v-click>

<v-click>

```bash
# Combiner les deux
git diff -w --color-moved
```

Parfait pour les refactorings : on voit immédiatement ce qui bouge vs ce qui change vraiment.

</v-click>

<!--
color-moved : disponible depuis git 2.15 (2017). Beaucoup de devs ne savent pas que ça existe.
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

</v-click>

<v-click>

> Les deux transforment radicalement la lisibilité des diffs. À tester absolument.

</v-click>

<!--
Montrer une capture d'écran ou une démo si possible. L'impact visuel est immédiat.
delta : https://github.com/dandavison/delta
diff-so-fancy : https://github.com/so-fancy/diff-so-fancy
-->

---
layout: section
---

# Naviguer plus vite

Moins taper, aller plus loin

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

**Maintenant (git 2.23+)**

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

<v-clicks>

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

```bash
# Cas concret : hotfix rapide
git switch -c hotfix/urgent
# ... correction ...
git switch main
git merge -           # merger hotfix depuis main
git switch -          # retour sur sa branche de feature
```

</v-clicks>

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
```

<v-clicks>

```bash
# -vv : voir aussi le tracking remote
git branch -vv
```

```
  feature/auth  a3f2e1c [origin/feature/auth] Add JWT middleware
* main          c1e9a4b [origin/main: behind 3] Merge pull request #42
```

```bash
# Trier par date de dernier commit
git branch --sort=-committerdate -v
```

</v-clicks>

<!--
branch -vv révèle aussi les branches qui ont divergé du remote. Très utile en équipe.
-->

---
layout: section
---

# Réécrire l'historique

Un historique propre, c'est un cadeau pour vos collègues

---
layout: default
---

# Rebase interactif

Réécrire les N derniers commits avant de pousser.

```bash
git rebase -i HEAD~5
```

<v-clicks>

```
pick a1b2c3d Ajouter le formulaire de connexion
pick b2c3d4e WIP
pick c3d4e5f fix typo
pick d4e5f6g Ajouter les tests
pick e5f6g7h Correction review
```

Commandes disponibles :

| Commande | Action |
|----------|--------|
| `pick` / `p` | Garder le commit tel quel |
| `reword` / `r` | Modifier le message |
| `edit` / `e` | Modifier le contenu |
| `squash` / `s` | Fusionner avec le précédent (garder msg) |
| `fixup` / `f` | Fusionner avec le précédent (supprimer msg) |
| `drop` / `d` | Supprimer le commit |

</v-clicks>

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
git config --global rebase.instructionFormat "%s (%an)"
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
p a1b2c3d Ajouter le formulaire de connexion (Alice)
p b2c3d4e WIP (Bob)
s c3d4e5f fix typo (Alice)
```

</v-click>

<v-click>

Sur une branche de feature à plusieurs, l'auteur évite les mauvaises surprises.

</v-click>

---
layout: two-cols
---

# fixup et autosquash

Corriger un vieux commit **proprement**.

**Workflow classique (laborieux) :**
```bash
git rebase -i HEAD~5
# éditer manuellement, déplacer les lignes
# changer pick en fixup
```

**Workflow avec fixup :**

```bash
# 1. Créer un commit de correction ciblé
git add -p
git commit --fixup a1b2c3d

# Résultat dans le log :
# a1b2c3d Ajouter le formulaire
# f2e1d0c fixup! Ajouter le formulaire  ← nouveau
```

::right::

<v-click>

```bash
# 2. Rebase avec autosquash
git rebase -i --autosquash HEAD~6
```

Git positionne et marque automatiquement le fixup au bon endroit :

```
pick a1b2c3d Ajouter le formulaire
fixup f2e1d0c fixup! Ajouter le formulaire
pick b2c3d4e Suite du travail
```

Plus qu'à sauvegarder. ✅

</v-click>

<v-click>

```bash
# Activer autosquash par défaut
git config --global rebase.autoSquash true
```

</v-click>

<!--
--fixup accepte aussi un message partiel : git commit --fixup :/formulaire
-->

---
layout: two-cols
---

# `rebase --onto`

Transplanter des commits d'une base vers une autre.

**Syntaxe :**
```bash
git rebase --onto <nouvelle-base> <ancienne-base> [<branche>]
```

**Cas d'usage typique :**
```
main ── A ── B
              └── feature ── C ── D ── E
                              └── sous-feature ── F ── G
```

```bash
# sous-feature a été branchée depuis feature
# On veut la rebaser directement sur main
git rebase --onto main feature sous-feature
```

::right::

<v-click>

**Résultat :**
```
main ── A ── B ── F' ── G'
              └── feature ── C ── D ── E
```

</v-click>

<v-click>

**Autre cas : supprimer des commits au milieu**
```bash
# Supprimer les commits C et D de feature
git rebase --onto B D feature
#           ^        ^
#       nouvelle   dernier commit
#       base       à supprimer
```

</v-click>

<v-click>

> `--onto` = "prends ces commits, pose-les ailleurs". Puissant mais demande de visualiser l'arbre.

</v-click>

<!--
Très utile quand on a fait une branche depuis la mauvaise base.
Mnémotechnique : --onto <destination> <exclure-depuis> <branche>
-->

---
layout: default
---

# `git history` — commandes expérimentales

<div class="text-orange-400 font-bold mb-4">⚠️ Expérimental — git 2.44+ requis</div>

Deux nouvelles commandes de haut niveau pour réécrire l'historique.

<v-clicks>

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

</v-clicks>

<v-click>

**Pourquoi c'est intéressant ?**

Ces commandes font la même chose que le rebase interactif mais avec une interface plus directe. `reword` en particulier évite d'ouvrir toute la liste des commits juste pour modifier un message.

> Ces commandes sont encore en développement. L'API peut changer.

</v-click>

<!--
Introduites dans git 2.44 (février 2024). À surveiller pour les prochaines versions.
Vérifier disponibilité : git history --help
-->

---
layout: section
---

# Travailler en parallèle

Plusieurs branches, sans stash ni panique

---
layout: two-cols
---

# Worktrees

Plusieurs working trees pour **un seul dépôt**.

**Le problème :**
```bash
# On est sur feature/big-refactor depuis 3 jours
# Un bug urgent arrive sur main
git stash          # stasher tout
git switch main    # changer de branche
# ... corriger ...
git switch feature/big-refactor
git stash pop      # espérer que ça se passe bien
```

😰 Contexte perdu, risque de conflit

::right::

<v-click>

**Avec les worktrees :**
```bash
# Créer un worktree pour le hotfix (dossier séparé)
git worktree add ../hotfix main

# Aller corriger dans le nouveau dossier
cd ../hotfix
# ... corriger, committer, pousser ...

# Revenir sans avoir rien touché
cd ../git-cli
# feature/big-refactor intacte, aucun stash
```

</v-click>

<v-click>

```bash
git worktree list          # voir tous les worktrees
git worktree remove ../hotfix   # nettoyer
```

> Idéal aussi pour : review de PR, comparaison de deux branches côte à côte.

</v-click>

<!--
Les worktrees partagent le même .git. Pas besoin de re-cloner.
On ne peut pas avoir la même branche dans deux worktrees simultanément.
-->

---
layout: section
---

# Sécurité et fiabilité

Éviter les catastrophes

---
layout: default
---

# `push --force-with-lease`

`git push --force` devrait presque toujours être remplacé.

<v-clicks>

**Le problème avec `--force` :**
```bash
# Alice et Bob travaillent sur la même branche
# Bob a poussé un commit pendant qu'Alice rebasait
git push --force   # ← Alice écrase le commit de Bob silencieusement
```

**La solution :**
```bash
git push --force-with-lease
# Vérifie que le remote correspond à ce qu'on a fetché
# Si quelqu'un a poussé entre-temps → REJET avec erreur claire
```

```bash
# Encore plus strict (git 2.30+)
git push --force-with-lease --force-if-includes
# Vérifie que le remote est dans notre historique local
```

</v-clicks>

<v-click>

```bash
# Alias indispensable
git config --global alias.pf "push --force-with-lease"
git pf
```

</v-click>

<!--
force-with-lease existe depuis git 1.8.5 (2013). Pas d'excuse pour utiliser --force.
force-if-includes : protection supplémentaire si le fetch a été fait mais pas intégré.
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

<v-clicks>

**Cas d'usage typique :**
```bash
# Branche longue qu'on rebase régulièrement sur main
git rebase main    # → conflit sur src/Service/Auth.php
# Résoudre le conflit manuellement
# rerere enregistre la résolution

# La semaine suivante, même rebase
git rebase main    # → même conflit
# rerere rejoue automatiquement la résolution ✅
```

```bash
# Voir les résolutions enregistrées
git rerere status
git rerere diff    # voir ce que rerere va appliquer
```

</v-clicks>

<!--
rerere.autoUpdate : applique la résolution ET indexe le fichier. Sans ça, il faut faire git add manuellement.
Très utile sur les projets où une branche de feature vit longtemps en parallèle de main.
-->

---
layout: section
---

# Configuration

Mettre en place une fois, en profiter partout

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
# macOS
.DS_Store
.AppleDouble

# JetBrains IDE
.idea/
*.iml

# VS Code
.vscode/
*.code-workspace

# Vim/Neovim
*.swp
*.swo
.netrwhist

# Fichiers locaux
*.local
.env.local
```

</v-click>

<v-click>

> Règle simple : si c'est lié à **votre machine ou votre outil**, ça va dans le global. Si c'est lié au **projet**, ça va dans le `.gitignore` du projet.

</v-click>

<!--
Évite les PR polluées par des fichiers .DS_Store ou .idea.
À faire une fois et ça s'applique à tous vos projets.
-->

---
layout: section
---

# Aliases

Taper le moins possible

---
layout: default
---

# Pourquoi des aliases ?

<v-clicks>

`git status` → **11 caractères**, 50 fois par jour = **550 caractères** pour rien

`git s` → **5 caractères** ✅

</v-clicks>

<v-click>

**Objectif :** max 3-4 caractères après `git` pour les commandes fréquentes.

```bash
# Voir ses aliases actuels
git config --list | grep alias

# Savoir d'où vient chaque config
git config --list --show-origin
```

</v-click>

<v-click>

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

<v-click>

> Certains frameworks (Oh My Zsh, Fish) incluent déjà des alias git. Vérifier avant de dupliquer.

</v-click>

<!--
Oh My Zsh git plugin : https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/git
gst = git status, gco = git checkout, etc.
-->

---
layout: center
class: text-center
---

# En résumé

<v-clicks>

Ces commandes existent depuis des années.

La plupart des développeurs ne les connaissent pas.

</v-clicks>

<v-click>

**Aujourd'hui, choisissez 2-3 choses à adopter :**

- `git add -p` pour des commits plus propres
- `push --force-with-lease` pour ne plus jamais écraser le travail d'un collègue
- `rerere.enabled` si vous rebasez souvent
- delta pour rendre les diffs agréables à lire
- 3-4 aliases pour les commandes les plus fréquentes

</v-click>

<v-click>

**Pour aller plus loin :**
```bash
git help everyday   # commandes git du quotidien
man git-config      # toutes les options de configuration
```

</v-click>

<!--
Questions ?

Liens :
- delta : https://github.com/dandavison/delta
- diff-so-fancy : https://github.com/so-fancy/diff-so-fancy
- git documentation : https://git-scm.com/docs
-->
