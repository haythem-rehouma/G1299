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



# Annexe 1 




# 1. **Git Merge**

👉 **Ce que ça fait** :

* Il **fusionne** deux branches et garde l’historique tel quel.
* Résultat : un **commit de merge** est ajouté pour indiquer la fusion.

### Exemple visuel :

```
A---B---C  (master)
     \
      D---E  (feature)
```

Après un `git merge master` dans `feature` :

```
A---B---C  (master)
     \     \
      D---E---M  (feature)
```

(`M` = commit de merge)

✔ Avantages :

* On garde **toute l’histoire réelle**.
* Moins risqué, car on ne réécrit pas l’historique.

✘ Inconvénients :

* L’historique peut devenir compliqué (beaucoup de branches et de merges).



# 2. **Git Rebase**

👉 **Ce que ça fait** :

* Il **déplace/rejoue** tes commits au-dessus de la branche cible.
* Résultat : un historique **linéaire**, comme si tes commits avaient été créés après les derniers de `master`.

### Exemple visuel :

```
A---B---C  (master)
     \
      D---E  (feature)
```

Après un `git rebase master` dans `feature` :

```
A---B---C  (master)
          \
           D'---E'  (feature)
```

(`D'` et `E'` sont des copies de D et E, rejoués sur C)

✔ Avantages :

* Historique **propre et linéaire**.
* Plus facile à lire avec `git log`.

✘ Inconvénients :

* **Réécrit l’historique** → dangereux si les commits sont déjà poussés et partagés.
* Peut entraîner plus de conflits si beaucoup de commits à rejouer.



# 3. **Quand utiliser Merge ou Rebase ?**

### Utilise **Merge** quand :

* Tu veux garder une trace claire de **toutes les branches et fusions**.
* Tu travailles en équipe et tu veux éviter les risques liés à la réécriture de l’historique.
* Exemple : intégrer une feature dans `main` → `git merge`.

### Utilise **Rebase** quand :

* Tu veux un **historique linéaire et propre** (souvent demandé dans les projets open source).
* Tu es en train de travailler seul sur ta branche et tu veux la mettre à jour avec `main` avant de faire un merge final.
* Exemple : avant un pull request → `git rebase main`.



# 4. **Bonnes pratiques**

* Sur **ta branche personnelle** (pas encore poussée) → `git rebase` est parfait.
* Pour fusionner une **feature terminée dans main** → privilégie `git merge`.
* **Jamais rebase une branche déjà poussée** et partagée avec d’autres (ça casse l’historique).



👉 En résumé :

* **Merge** = sécurité + historique complet.
* **Rebase** = propreté + historique linéaire.





#  Tableau comparatif Git Merge vs Git Rebase

| Aspect                 | **Merge**                                                                                                                       | **Rebase**                                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Principe**           | Crée un **commit de merge** qui fusionne deux branches.                                                                         | **Rejoue** les commits de ta branche par-dessus une autre (réécrit l’historique).                         |
| **Historique**         | Conserve **toute l’histoire réelle** (branches + commits).                                                                      | Produit un **historique linéaire** et simplifié.                                                          |
| **Exemple visuel**     | <pre>A---B---C (master)<br> \      <br>  D---E---M (feature)</pre>`M` = commit de merge                                         | <pre>A---B---C (master)<br>        <br>         D'---E' (feature)</pre>`D'` et `E'` = commits rejoués     |
| **Avantages**          | - Plus sûr (pas de réécriture)<br>- Recommandé pour travail collaboratif<br>- Montre quand les branches ont divergé et fusionné | - Historique plus lisible<br>- Évite les commits de merge « bruit »<br>- Idéal pour des PR propres        |
| **Inconvénients**      | - Historique peut devenir complexe avec beaucoup de merges                                                                      | - Réécrit l’historique (dangereux si déjà partagé)<br>- Peut créer beaucoup de conflits si longue branche |
| **Quand l’utiliser ?** | - Fusionner une feature terminée dans `main`<br>- Collaborer sans casser l’historique                                           | - Mettre à jour ta branche perso avant merge/PR<br>- Nettoyer l’historique avant de pousser               |
| **Commande type**      | `git merge master`                                                                                                              | `git rebase master`                                                                                       |



#  Règle d’or simplifiée 

* **Avant de pousser ta branche** → tu peux utiliser **rebase** pour nettoyer.
* **Quand tu fusionnes dans `main` ou `develop`** → utilise **merge** pour éviter les problèmes.
* **Jamais rebase une branche déjà partagée** avec d’autres.



