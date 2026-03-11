---
title: Trous et boomerang
layout: default
use_mathjax: true
---

# Trous et boomerang

La semaine dernière, nous avons mis des ennemis sur le chemin de notre joueur, mais nous ne lui avons pas donné les moyens de se défendre !
Nous allons remédier à cela en lui fournissant un boomerang, qui lui permettra d'attaquer les ennemis à distance.
Nous allons aussi ajouter un peu de réalisme à notre joueur en le faisant marcher et se tenir dans 4 directions distinctes.
Finalement, nous allons ajouter des trous dans lesquels il lui faudra éviter de tomber.

À nouveau, la première section doit être faite en commun.
Puis, répartissez-vous le reste des tâches de manière à pouvoir travailler indépendemment et en parallèle.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Une classe et une direction pour le joueur

Maintenant que vous avez vu ce qu'est [l'héritage](/tutoriels/heritage-et-polymorphisme.html), vous allez pouvoir l'exploiter pour mieux structurer votre projet.
Vous avez problement beaucoup de code dans `GameView` qui s'occupe de différents choses : le joueur, les ennemis, etc.
Il vaudrait mieux séparer ces différents concepts.

Créez maintenant une nouvelle classe `Player` *qui hérite de* `arcade.TextureAnimationSprite` :

```python
class Player(arcade.TextureAnimationSprite):
    ...
```

À vous de juger, désormais, s'il est pertinent de créer des fichiers séparés ou pas (voir aussi le tutoriel sur les [modules](/tutoriels/modules.html)).

La classe `Player` pourra ajouter les attributs nécessaires à la gestion du joueur, notamment ses déplacements.
Remplacez les endroits qui manipulaient le sprite du joueur par l'usage de votre nouvelle classe `Player`.

Puisque le joueur devra pouvoir "regarder" dans 4 directions différentes, définissez un nouveau type `Direction` pouvant représenter une des 4 directions cardinales (nord, est, sud, ouest).
Ajoutez un attribut ou une propriété `direction` de type `Direction` dans la classe `Player`.
Pour l'instant, elle est toujours initialisée au sud, et n'est pas modifiée.

**Question de design :** Comment définissez-vous le type `Direction`, et pourquoi ?

Refactorisez le code de `GameView` qui manipule le mouvement du joueur.
Ajoutez une ou des méthodes dans la classe `Player` qui vont se charger des calculs, en fonction des touches de direction qui sont appuyées.

**Question de design :** Ces méthodes reçoivent-elles n'importe quel `symbol: int`, comme dans `on_key_press`, ou reçoivent-elles un type de données plus spécifique ? Pourquoi ce choix ?

Vérifiez que vos tests passent toujours.
Nous n'avons rien modifié en termes de fonctionnalités, mais il se peut que vous deviez ajuster légèrement le code de vos tests.

Remarque subsidiaire : peut-être y a-t-il un autre type de sprite dans votre jeu qui mériterait d'être refactorisé en une classe dédiée ?

## Trous

Ajoutez une nouvelle sorte de cellule sur nos map : le *trou* (*hole* en anglais).
Utilisez le caractère `O` (la lettre 'o' majuscule) dans les fichiers de maps.
La texture pour un trou peut être trouvée dans la grille de textures principales à la ligne 4, colonne 8 (`_overworld_grid[18*4 + 8]` si vous avez encore la nomenclature de la première semaine).

![Trou (hole)](/assets/img/projet/hole.png){: class="asset-scaled-to-32"}

Le trou n'est pas un obstacle comme les buissons.
On ne veut pas empêcher le joueur d'aller dessus.
Par contre, s'il vient effectivement "trop près" au-dessus du trou, il tombe et perd (comme s'il avait été touché par un monstre).

Par "trop près", on entend précisément : si le distance entre la position du joueur et celle d'un trou est inférieure ou égale à 16 (la moitié de la taille en pixels d'une cellule).
Le joueur pourra donc se trouver *partiellement* au-dessus d'un trou sans tomber.
Mais s'il se retrouve trop près de son centre, alors il tombe.

Vérifiez que le joueur peut marcher entre deux trous séparés par une cellule d'herbe.
À l'inverse, vérifiez que s'il y a deux trous l'un à côté de l'autre, il n'est pas possible de se glisser au milieu d'eux.

### Petit challenge

Idéalement, on voudrait éviter de tester la distance entre le joueur et *chaque* trou de la map.
Si $n$ est le nombre de trous sur la map, peut-on détecter la chute dans un des $n$ trous en $\Theta(1)$ plutôt qu'en $\Theta(n)$ ?

Indice : Pourrait-on faire une "pré-sélection" de maximum quelques trous candidats avec les outils que nous donne Arcade ?

## Affichage du score

Votre joueur ramasse des crystaux, mais pour l'instant, rien ne montre combien il en a.
On veut ajouter un compteur de score sur notre jeu.

Le score doit toujours refléter le nombre de crystaux qu'a ramassés le joueur depuis le début du jeu.
S'il perd, le compteur est remis à zéro.

On veut voir apparaître le score à un emplacement *fixe* de l'écran, par exemple, en haut à gauche.
Lorsque la caméra se déplace pour suivre le joueur, ces éléments de "UI" (user interface) ne doivent pas se déplacer.

Le moyen le plus simple d'obtenir cet effet est de définir une *deuxième* caméra.
Vous ne déplacerez pas cette caméra ; elle restera à sa configuration d'origine, dont la position $(0, 0)$ est en bas à gauche de l'écran.
Dessinez les éléments de UI dans un second bloc `with ... activate():`, utilisant cette nouvelle caméra.

Pour dessiner du texte à l'écran, utilisez [`arcade.Text`](https://api.arcade.academy/en/latest/api_docs/api/text.html).

## Orientation du joueur

Le joueur lancera le boomerang (voir section suivante) dans la `direction` vers laquelle il regarde.
On voudra donc changer cette direction en fonction des touches qui sont appuyées.
De plus, on voudra visualiser la direction dans lequelle il regarde, en choisissant une animation différente en fonction de cette direction.

Les assets qui vous ont été fournis contiennent des animations pour les quatre directions cardinales :

* Sud : `char_idle_down_anim_strip_6.png` (que vous utilisez déjà)
* Nord : `char_idle_up_anim_strip_6.png`
* Ouest : `char_idle_left_anim_strip_6.png`
* Est : `char_idle_right_anim_strip_6.png`

Remarquez que le schéma est toujours le même : `char_idle_{dir}_anim_strip_6.png` où `{dir}` est parmi `down`, `up`, `left` et `right`.

Vous pouvez changer l'animation d'une `TextureAnimationSprite` (et donc de votre `Player`) pendant le jeu en modifiant sa propriété [`animation`](https://api.arcade.academy/en/stable/api_docs/api/sprites.html#arcade.TextureAnimationSprite.animation).

Mais comment décider dans quelle direction regarde le joueur ?
Les règles suivantes doivent s'appliquer :

* si la flèche du bas (↓) est enfoncée, alors il regarde vers le sud ;
* sinon la flèche du haut est envondée, alors il regarde vers le nord ;
* sinon, de la même manière vers l'ouest, ou vers l'est ;
* sinon (aucune des 4 flèches n'est enfoncée), il *continue de regarder dans la même direction qu'avant*.

Exemples :

* Après s'être déplacé vers la droite, même une fois arrêté, le joueur continue donc de regarder vers la droite.
* Pendant qu'il se déplace en diagonale nord-est, il regarde vers le nord (pas vers l'est).

### Optionnel : encore un peu plus de réalisme

Les assets fournis contiennent une seconde série de 4 animations.
Celles que nous avons utilisées jusqu'à présent sont prévues pour être utilisées lorsque le joueur ne bouge pas.
Une autre série, suivant le modèle `char_run_{dir}_anim_strip_6.png`, est prévue pour les cas où le joueur se déplace.

Ajustez encore le joueur pour prendre en compte ce deuxième aspect.
Choisissez la bonne série d'animations en fonction de si le joueur se déplace ou pas ; en plus de sa direction.

## Boomerang

Le gros morceau de la semaine est bien entendu l'ajout du boomerang.
Le joueur va pouvoir lancer son boomerang dans sa direction actuelle en appuyant sur la touche `D` du clavier.
Le boomerang se déplace en ligne droite jusqu'à toucher un obstacle ou un monstre, après quoi il revient vers le joueur.

L'animation que vous pouvez utiliser pour le boomerang est `"assets/provided/boomerang-sheet.png"`.
Elle possède 8 frames, chacune de 25 millisecondes (et non pas les 100 ms par défaut !).
Voir le paramètre `frame_duration` de la fonction `_load_animation_strip`.

Nous suggérons une vitesse de 8 pixels par frame pour le boomerang.

Les règles suivantes doivent s'appliquer :

Le boomerang a 3 états possible : inactif (*inactive*), lancement (*launching*) et retour (*returning*).
Il est inactif au début du jeu.
Lorsqu'il est inactif, le boomerang n'apparaît pas sur le jeu.

Lorsque le joueur appuie sur la touche `D`, et seulement si le boomerang est actuellement inactif, alors le joueur lance le boomerang.
Le boomerang passe dans l'état "lancement", et part dans la direction dans laquelle regarde le joueur, depuis la position actuelle du joueur.
Il continue en ligne droite jusqu'à ce que l'une des choses suivantes se produisent :

* il a parcouru la distance équivalent à 8 cellules ;
* il a touché un obstacle (un buisson) ;
* il a touché un monstre.

Dans ce dernier cas, le monstre est vaincu et retiré du jeu (comme quand on ramasse un crystal).

Dans tous les cas, le boomerang passe alors dans l'état "retour".
Il cherche alors à retourner au joueur.

* À chaque frame, il se déplace *vers le joueur*, même si le joueur s'est déplacé depuis qu'il a lancé le boomerang.
  Il peut donc avoir une trajectoire courbe à travers plusieurs frames, si le joueur se déplace.
* Il n'est *pas arrêté* par les obstacles ni les monstres.
* Mais il tue quand même les monstres rencontrés sur son chemin !

Il s'arrête alors quand il est "suffisamment proche" du joueur (à vous de définir ce que cela veut dire).
Il repasse dans l'état "inactif" (et n'est donc plus visible), jusqu'à son prochain lancement.

**Questions de design :**

* Avez-vous défini une classe séparée pour gérer le boomerang, et si oui, étend-elle une classe de sprite ?
  Pourquoi ?
* Comment gérez-vous les 3 états du boomerang ?

## Compléter le log et les réponses

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).
