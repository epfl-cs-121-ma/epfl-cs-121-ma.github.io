---
title: Setup rapide d'un projet
layout: default
---

# Setup rapide d'un projet

Cette petite référence rappelle les instructions de création d'un nouveau projet.
Vous en aurez besoin à chaque séance d'exercices, ainsi que pour votre gros projet.

## Création du projet en tant que tel

⚠️⚠️⚠️ Sur **Windows** : faites attention à ne *jamais* travailler dans un dossier géré par OneDrive.
OneDrive va entrer en conflit sévère avec `uv` et `git`.
Des tas de choses vont mal se passer dans une ou deux semaines si vous faites ça. ⚠️⚠️⚠️

```
$ pwd
.../projets
$ mkdir nouveau-projet
$ cd nouveau-projet
$ uv init -p 3.14
```

⚠️ Vérifiez toujours qu'un fichier `.gitignore` a bien été créé par `uv init`.
Vous pouvez par exemple exécuter la commande `ls .gitignore` et vérifier qu'elle n'affiche pas d'erreur.
Si ce n'est pas le cas, c'est que vous avez accidentellement créé votre projet *à l'intérieur d'un autre repository git*.
Des tas de choses de choses vont mal se passer dans le futur si c'est le cas ! ⚠️

Ajouter à la fin de `pyprojet.toml` :

```
[tool.ruff.lint]
select = ["ANN"]
```

Intaller ty, Ruff et pytest :

```
$ uv add --dev ty ruff pytest
```

Créer le dossier `tests` et son fichier `__init__.py` :

```
$ mkdir tests
$ touch tests/__init__.py
```

Ajouter `-> None` dans la définition de `def main` dans `main.py` :

```diff
-def main():
+def main() -> None:
     print("Hello from solutions-serie-02!")
```

Vérifiez que ty, Ruff et pytest s'exécutent sans erreur :

```
$ uv run ty check
$ uv run ruff check
$ uv run pytest
```

## Configuration git

(Optionnel)

Faire un commit avec l'état initial, avant de commencez votre travail :

```
$ git add .
$ git commit -m "Initial commit."
```

Créer un nouveau repo *privé* sur GitHub.

⚠️ Ne *pas* cocher la case "Add a readme file", ne *pas* sélectionner de "`.gitignore` template" ni de licence à ce stade. ⚠️

Une fois créé, récupérer l'URL SSH `git@...` du repo.
Connecter le nouveau projet à ce repo et pousser le commit initial.

```
$ git remote add origin git@...
$ git push -u origin main
```
