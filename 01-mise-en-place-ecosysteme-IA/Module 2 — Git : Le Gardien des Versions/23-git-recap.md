# 01 – `merge` et `rebase`

* **merge** : permet de fusionner deux branches. Il crée un commit de fusion qui garde l’historique complet des deux branches.
* **rebase** : permet de réappliquer les commits d’une branche par-dessus une autre, ce qui donne un historique plus linéaire, sans commit de merge.



# 02 – `squash`

* **squash** : consiste à « écraser » plusieurs commits en un seul. C’est pratique pour nettoyer l’historique avant de pousser sur un dépôt (ex. réduire 10 petits commits de test en 1 seul commit clair).



# 03 – `reset`

* **reset** : déplace le pointeur `HEAD` vers un autre commit et ajuste la zone de staging et/ou le répertoire de travail.

  * `--soft` : garde les modifications en staging.
  * `--mixed` (par défaut) : garde les fichiers modifiés mais les retire du staging.
  * `--hard` : supprime toutes les modifications locales (dangereux).



# 03 – `reset` (exercice pratique)

* Exercices pour comprendre les différents modes de `reset` et leurs effets sur l’historique et les fichiers en cours.



# 04 – `revert`

* **revert** : crée un nouveau commit qui annule un commit précédent.
  ➝ Contrairement à `reset --hard`, il ne supprime pas l’historique, il ajoute un correctif.



# 05 – `.gitignore`

* **.gitignore** : fichier spécial qui liste les fichiers/dossiers à ne pas suivre par Git (ex. fichiers temporaires, binaires, `.env`, etc.).



# 06 – `diff`

* **diff** : affiche les différences entre deux versions de fichiers (entre le répertoire de travail, le staging et/ou les commits).
  ➝ Très utile pour voir ce qui a changé avant un `commit`.



# 07 – `sync` avec GitHub

* **git pull + git push** : synchroniser son dépôt local avec le dépôt distant (GitHub).
* `git fetch` : télécharge les changements mais sans les fusionner.
* `git pull` : `fetch` + `merge` ou `rebase`.



# 08 – `branches`

* **branch** : crée, liste ou supprime des branches.
* `git branch nouvelle-branche` → créer une nouvelle branche.
* `git checkout -b nouvelle-branche` → créer **et** basculer dessus.



# 09– `remote branches`

* **remote** : gère les dépôts distants.
* `git remote -v` → liste les dépôts distants.
* `git push origin branche` → envoie une branche vers GitHub.
* `git fetch origin` → récupère les branches distantes sans fusion.



# 10 – `cherry-pick`

* **cherry-pick** : applique un commit spécifique d’une branche sur une autre.
  ➝ Exemple : « je veux juste prendre ce correctif précis, sans tout fusionner ».



# 11 – `stash`

* **stash** : met de côté les modifications en cours (comme une pile temporaire), pour travailler sur autre chose.
* `git stash` → sauvegarde et nettoie le répertoire de travail.
* `git stash pop` → restaure la dernière sauvegarde.



# 12 – `worktree`

* **worktree** : permet de travailler sur plusieurs branches en parallèle dans différents dossiers de travail liés au même dépôt.
  ➝ Pratique pour tester deux branches côte à côte sans cloner deux fois.



# 13 – `tags`

* **tag** : associe un nom lisible à un commit précis (souvent pour marquer une version).
* `git tag v1.0` → crée un tag local.
* `git push origin v1.0` → pousse le tag vers GitHub.



# 14 – `merge-rebase`

* Étude de cas pour comparer en détail `merge` et `rebase` :

  * `merge` garde la trace de toutes les branches.
  * `rebase` réécrit l’historique pour le rendre linéaire.

