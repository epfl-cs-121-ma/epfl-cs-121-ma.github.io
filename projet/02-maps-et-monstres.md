---
title: Maps et monstres
layout: default
use_mathjax: true
---

# Maps et monstres

Cette fois, le projet est lancé pour de bon.
Cette semaine, nous vous demandons d'ajouter deux fonctionnalités à votre jeu.
D'une part, au lieu de "hard-coder" un peu d'herbe et des buissons dans votre `__init__()`, vous devrez *charger* la map (carte) depuis un fichier.
D'autre part, vous allez ajouter des monstres pour mettre des bâtons dans les roues du joueur : de petits *spinners* qui se déplacent sur l'herbe en ligne droite.

À ce stade-ci du projet, nous espérons fortement que vous avez trouvé votre binôme.
La première section doit être faite en commun.
Puis, répartissez-vous le reste des tâches de manière à pouvoir travailler indépendemment et en parallèle.

Nous rappelons qu'il est toujours nécessaire [d'écrire des tests](./01-decouverte.html#tests) pour les tâches qui vous sont demandées.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Une structure de données pour la map

Cette section est à faire en commun.

On va devoir charger la map depuis un fichier, faire certains calculs sur celles-ci (pour les spinners), et finalement créer les bons sprites.
On souhaite avoir une bonne *structure de données* pour représenter la map *de manière abstraite*, afin de faciliter les différentes opérations sur celle-ci, et de les rendre plus sûres (moins susceptibles d'être buggées).
Cette structure doit être à la fois découplée du format textuel des fichiers que de la réalisation concrète avec des sprites.
Idéalement, cette structure sera totalement découplée d'Arcade, afin qu'on puisse tester certains des calculs sans utiliser du tout de `Window`.

Cette structure sera un moyen de communication entre les différents morceaux de votre projet.

* La lecture de carte consommera un fichier texte et produira une map.
* Votre `GameView` consommera une map pour créer les sprites au début du jeu.
* Votre code pour les spinners consommera une map pour calculer les limites de leurs déplacements.
* Vos tests définiront des maps isolées, minimales pour leurs besoins.

### Classe `Map` et ses `GridCell`s

Dans un nouveau fichier `map.py`, définissez une classe `Map` qui représentera une map de votre jeu.
Une map est une *grille* $w \times h$ de "cellules" (`GridCell`).
Les coordonnées sont des *entiers*, de sorte qu'il y a $w \cdot h$ cellules dans une map, indicées par des couples $(x, y)$ d'entiers avec $0 \leq x < w$ et $0 \leq y < h$.

**Une `Map` est *immuable*.**
Veillez à ce que la classe `Map` garantisse cet aspect avec de l'encapsulation appropriée et/ou l'usage adéquat de `Final`.

Définissez au moins l'interface publique suivante :

* Des attributs entiers `width` et `height`.
* Des attributs entiers `player_start_x` et `player_start_y`.
* Une méthode `get(x: int, y: int)` qui renvoie une `GridCell`.
  À vous de décider ce qu'elle doit faire si le couple $(x, y)$ est hors des bornes de la grille.

Une `GridCell` représente un des choix possibles pour chaque cellule.
Pour l'instant (à l'issue de la première étape de découverte), 3 choix sont possibles :

* de l'herbe (ou "rien", si vous préférez) ;
* un buisson ;
* un cristal.

Puisqu'il y a un nombre fini de possibilités, veillez à **définir `GridCell` comme un `Enum`**.

### Refactoring : `GameView` consomme une `Map`

Dans `map.py`, définissez une constante globale `MAP_DECOUVERTE` de type `Map`.
Elle doit représenter la structure (abstraite) de la carte que vous aviez créée à la main dans `__init__` la semaine passée.

Modifiez le `__init__` de votre `GameView` pour prendre un paramètre `map: Map`.
Au lieu de hard-coder la création des sprites (et le calcul de la taille du monde), votre `__init__` doit maintenant inspecter la `map` pour créer les sprites adéquats.

* Calculer la taille du monde (`world_width` et `world_height`) en fonction de `map.width` et `map.height`.
* Créer un sprite d'herbe à toutes les positions $(x, y)$ dans les bornes de la map.
* Utiliser `map.get(x, y)` à toutes les positions pour savoir s'il faut y créer un sprite supplémentaire et lequel (rien, un buisson ou un cristal).
* Créer et placer le sprite du joueur en fonction de `map.player_start_x` et `map.player_start_y`.

Dans `main.py` et dans vos tests, lorsque vous instanciez `GameView`, donnez-lui `MAP_DECOUVERTE` en paramètre.

Vérifiez que vos tests tournent toujours.
En principe, *on n'a rien changé* au comportement de votre programme.
On a seulement amélioré "l'intérieur" afin de faciliter les étapes suivantes du projet.

## Charger la map depuis un fichier

Jusqu'à présent, votre code de création de la map est "hard-codé".
C'est du code Python qui crée la grille avec des cellules données, placées à des coordonnées fixes.
C'est déjà fastidieux pour quelques buissons et cristaux.
Imaginez ce que ce sera si vous devez concevoir une carte plus grande par vous-mêmes.

Il faut donc que nous puissions *charger* une map *depuis un fichier*.
De cette manière, nous pourrons éditer la map séparément du code.

Pour les tests, nous aimerions aussi pouvoir intégrer directement le contenu d'un fichier de map dans une string Python.
Cela permettra à chaque test d'avoir sa map dédiée, visible au même endroit dans le code que le test lui-même.

### Format du fichier

Nous utiliserons un petit format de fichier simple, qui est plus ou moins visuel.
Cela vous permettra de concevoir votre propre map sans trop de difficultés, et sans avoir besoin de logiciel supplémentaire.
Voici un exemple du format que nous vous demandons de supporter :

```
width: 40
height: 15
---
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
x  *               S     x             x
x                        x    *   *    x
x                      * x             x
x         s        xxxxxxx            sx
x     xxx *   xx                       x
x                            xxx       x
x            xxxxxxxxxx        x       x
x         *                    x   *   x
x      S              S     *  x       x
x  *            *                      x
x                                      x
x     xx     s        xs               x
x  P      *           x       *        x
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
---
```

Enregistrez cette map dans le fichier `maps/map1.txt`.

Le fichier représente une `Map` en mode "texte".
Le format se compose de deux sections.

La première est la configuration.
On doit y trouver au moins la taille de la map, en nombre de cases de la grille, que nous noterons $(w, h)$.
La taille est donnée par une `width` et une `height`.
Les lignes de configuration doivent suivre exactement le format `key: value` (une clef `key`, suivi des caractères `:` et espace, suivi de la valeurr `value`).
La `width` et la `height` doivent être des entiers strictement positifs, exprimés en base 10.

La configuration s'achève avec exactement la ligne

```
---
```

Ensuite vient la carte à proprement parler.
Elle est composée de $h$ lignes.
Après les $h$ lignes vient à nouveau une ligne de terminaison `---`.

Chacune des $h$ lignes représente une ligne de la grille de la map.
La première ligne correspond à l'indice $h-1$ ; la dernière à l'indice $0$ (de sorte que visuellement les $y$ aillent vers le haut, comme dans Arcade).
Elle est constituée de $0$ à $w$ caractères (inclus) ; chaque caractère représentant une case de la grille (les indices vont de $0$ à $w-1$ de gauche à droite), et le type de cellule qui doit s'y trouver.
Le caractère espace (`' '`) indique l'herbe/le "rien".
La ligne peut contenir moins de $w$ caractères ; dans ce cas on suppose que les caractères "manquants" sont tous des espaces.
Une ligne peut être vide.
S'il y plus de $w$ caractères, c'est une erreur.

### Types de cellule

Les caractères ont la signification suivante :

| Caractère    | Type de cellulue   |
|--------------|--------------------|
| ` ` (espace) | Herbe/rien         |
| `x`          | Buisson            |
| `*`          | Cristal            |
| `s`          | Spinner horizontal |
| `S`          | Spinner vertical   |
| `P`          | Départ             |

Le `P` est un peu particulier.
Il représente la position de départ du joueur.
Il doit y en avoir exactement un sur la map (sinon, c'est une erreur).

Les `*` sont les cristaux à collecter.
Les spinners sont décrits dans les sections suivantes.

### Votre tâche

Pour cet aspect de votre projet, vous devez :

* Définir un type d'exception spécifique pour les erreurs dans les fichiers:

  ```python
  class InvalidMapFileException(Exception):
      pass
  ```

* Lire et décoder un fichier de carte tel que décrit ci-dessus, ou une string Python contenant la même chose, pour en faire une `Map`.
  En cas d'erreur de format, lever une `InvalidMapFileException`.
* Adapter votre `main` et vos tests pour utiliser cette nouvelle capacité, au lieu de toujours utiliser `MAP_DECOUVERTE`.
* Si une exception `InvalidMapFileException` a été levée, votre `main` doit émettre un message d'erreur à l'attention de l'utilisatrice ou l'utilisateur :
    * soit dans la console (avec `print()`), auquel cas le programme doit s'arrêter immédiatement ;
    * soit sous forme d'un écran dédié dans la fenêtre de jeu.

  Dans un cas comme dans l'autre, le message d'erreur devra être "user-friendly" (ce ne peut pas être une Traceback).

Vous devez pouvoir écrire des tests pour la lecture de fichier *indépendamment d'Arcade*.
Vérifiez qu'avec une entrée textuelle donnée (idéalement dans une string Python), vous obtenez la bonne `Map`.
Vérifiez que l'exception appropriée est levée en cas d'erreur de format.

Modifiez votre `main` de sorte qu'on puisse donner le nom du fichier de map en argument de votre programme.
Si on lance `uv run main.py maps/votre-map.txt`, vous devez charger la map stockée dans le fichier `maps/votre-map.txt`.
Si on lance seulement `uv run main.py`, vous devez charger `maps/map1.txt` par défaut.

Vous pouvez avoir accès aux arguments du programme avec `sys.argv`.
Lisez [sa documentation](https://docs.python.org/3/library/sys.html#sys.argv), et essayez :

```python
import sys

print(sys.argv)
```

pour voir comment l'utiliser.

## Monstres spinners

Les *spinners* sont de petits monstres métalliques qui se déplacent uniquement en ligne droite (horizontale ou verticale).
Quand ils rencontrent un obstacle (pour l'instant, un buisson), ils repartent dans l'autre sens.
Si le joueur touche un spinner, il perd immédiatement.

L'asset des spinners se trouve dans

```
assets/Top_Down_Adventure_Pack_v.1.0/Enemies_Sprites/Spinner_Sprites/spinner_run_attack_anim_all_dir_strip_8.png
```

L'animation comporte 3 frames (et non pas 8, comme le laisse supposer le nom de fichier !).

![Spinner](/assets/img/projet/spinner_run_attack_anim_all_dir.gif){: class="asset-scaled-to-32"}

### Règles

Un spinner est soit *horizontal* (`s`), soit *vertical* (`S`).
Il se déplace à vitesse constante dans la direction donnée.
Au départ, un spinner horizontal se déplace vers la droite ; un spinner vertical se déplace vers le haut.
Il change de direction lorsqu'il touche un obstacle.

Rien d'autre n'affecte les spinners.
Il ne sont pas arrêtés par d'autres spinners, d'autres cristaux, ou même le joueur.
Deux spinners peuvent donc se superposer ou se croiser.

Puisque seules la position initiale du spinner et les positions des obstacles (fixes) déterminent le mouvement d'un spinner, il est possible, **et nous vous demandons de le faire**, de calculer les limites gauche/droite ou bas/haut une fois pour toutes, au démarrage du jeu.

Nous suggérons une vitesse de 3 pixels par frame pour le déplacement des spinners.

Lorsque le joueur touche un spinner, il perd immédiatement.
Le jeu est alors réinitialisé, comme lorsqu'on appuie sur la touche Escape (voir semaine passée).

### Votre tâche

Pour cet aspect de votre projet, vous devez implémenter les spinners, avec les règles décrites ci-dessus.
En particulier, nous vous demandons de calculer les limites de déplacement d'un spinner *indépendamment d'Arcade*, en travaillant uniquement sur une `Map` et la position initiale du spinner.
Vous devez pouvoir écrire un test pytest qui teste ces calculs *sans dépendre d'Arcade*.

Par ailleurs, vous devez implémenter les règles des spinners :

* leur déplacement ;
* leur changement de direction quand ils atteignent leurs limites (calculées une fois pour toutes) ;
* leur effet sur le joueur (perte de la partie en cas de collision avec leur joueur).

## Liens utiles

Surtout pour la lecture du fichier :

* [Méthodes de la classe `str`](https://docs.python.org/3/library/stdtypes.html#string-methods).
* [La fonction `int`](https://docs.python.org/3/library/functions.html#int) permet de convertir une chaîne de caractères représentant un nombre en base 10 en un `int`.
* [Lecture de fichiers, C++ vs Python](/references/python-vs-cpp.html#fichiers) (rappel)
* [Lecture de fichiers, documentation officielle](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files)
* [La fonction `textwrap.dedent`](https://docs.python.org/3/library/textwrap.html#textwrap.dedent) permet d'écrire des chaînes littérales sur plusieurs lignes dans le code source, tout en ayant une indentation propre.
  Celle-ci sera **particulièrement utile pour vos tests**.
* [La fonction `pytest.raises`](https://docs.pytest.org/en/stable/reference/reference.html#pytest.raises) permet de tester qu'un bout de code lance bien l'exception qu'on veut.

Surtout pour les spinners :

* [Documentation de `arcade.Sprite`](https://api.arcade.academy/en/latest/api_docs/api/sprites.html#arcade.Sprite)
* Vous pourriez avoir envie d'utiliser les attributs `boundary_left`/`boundary_right` et `boundary_bottom`/`boundary_top` pour vos sprites de blobs ; leur valeur est ignorée par Arcade, donc vous en faites ce que vous voulez.

## Compléter le log et les réponses

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).

Il n'y a toujours pas de "question de design" cette semaine, puisque nous vous avons donné les clefs de designs les plus importantes (`Map`, `GridCell`, calcul des limites des spinners indépendamment d'Arcade).
