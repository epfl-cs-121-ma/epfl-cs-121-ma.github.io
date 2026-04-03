---
title: Blobs et interrupteurs
layout: default
use_mathjax: true
---

# Blobs et interrupteurs

Cette étape du projet est la plus longue.
De plus, la semaine de vacances et la semaine du midterm ne comptent pas dans le projet.
**Les prochaines instructions arriveront donc en semaine 11.**

Nous allons encore ajouter deux grandes fonctionnalités dans le jeu : des blobs au déplacement "intelligent", et des interrupteurs.
Les blobs se déplaceront aléatoirement au sol (en respectant les obstacles), mais tenteront d'attacker le joueur s'ils le voient.
Les interrupteurs pourront être activés ou désactivés pour ouvrir ou fermer des portails.

Dans les deux cas, vous aurez besoin de *bibliothèques externes*.
La semaine de vacances tombant tôt dans le semestre cette année, nous vous donnons cette série d'instructions plus tôt que prévu, en l'étalant plus.
Cependant, cela veut dire que vous devrez peut-être anticiper la lecture du [tutoriel sur les bibliothèques externes](/tutoriels/bibliotheques.html) (il est assez court).

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Blobs

Les chauves-souris et les spinners ne se soucient pas du joueur.
Ils se déplacent de manière aléatoire ou déterminée, mais c'est entièrement la faute du joueur s'il se jette sur leur passage.

Nous ajoutons maintenant un troisième type de monstre, le *blob* (*slime* en anglais).
Celui-ci se déplace au sol, de manière aléatoire tant qu'il ne "voit" pas le joueur.
Mais si le joueur entre dans sa ligne de vue (*line of sight* en anglais), alors il va tenter de le pourchasser.
Il utilise donc une forme d'IA élémentaire.

Comme le blob se déplace au sol, il doit éviter les obstacles !
Mais il ne déplace pas non plus simplement en ligne droite comme le spinner.
Il va donc devoir trouver un *chemin* (*path*) depuis sa position jusqu'à sa destination choisie.

⚠️ Une fois n'est pas coutume, je vous impose un interdit.
Vous n'avez **pas le droit** d'utiliser le [A-star path finding de Arcade](https://api.arcade.academy/en/stable/example_code/astar_pathfinding.html) pour réaliser les déplacements des blobs.
L'objectif de cette partie du projet est de vous rendre autonome sur l'utilisation de nouvelles bibliothèques, et d'intégrer des algorithmes un peu plus exigeants au sein de votre projet.
Arcade nous rend beaucoup de services pour ce projet, mais ici, d'un point de vue pédagogique, elle est un peu "too helpful".

### Caractéristiques

Un blob a une zone de patrouille, un peu comme la chauve-souris.
Nous vous proposons l'équivalent d'un carré de 7 cellules de côté, centré sur sa position de départ.
Ses *destinations aléatoires possibles* sont les cellules qui font partie de sa zone de patrouille et qui ne comportent pas d'obstacle (et qui sont dans les limites de la map).
Les obstacles d'un blob sont les buissons et les trous.
Remarquez que les destinations possibles d'un blob ne dépendent que de la `Map` et de sa position de départ.
Elles ne changeront pas pendant le jeu.

À tout moment, le blob a une "destination courante" qui est un point du jeu (des coordonnées en pixels).
Il se comporte de la façon suivante à chaque frame :

1. S'il peut *voir* le joueur (voir plus loin), alors il remplace sa destination courante par la position du joueur.
1. Sinon, s'il est arrivé suffisamment proche de sa destination (à quelques pixels près), il en choisit une nouvelle au hasard *parmi ses destinations aléatoires possibles*.
1. Dans un cas comme dans l'autre, il détermine un chemin vers sa nouvelle destination.
1. Puis (qu'il ait changé ou de destination ou pas), il se déplace le long de son chemin avec une vitesse *de norme constante* (nous suggérons 1 pixel par frame).

Dans un premier temps, nous vous recommandons d'ignorer le joueur.
Concentrez-vous d'abord sur le déplacement aléatoire.
On va maintenant vous expliquer comment on peut avoir un déplacement aléatoire qui respecte les obstacles.

### Chemins et navigation mesh

> **Hors cours** : pour une introduction plus "fun" en vidéo sur les navigation mesh'es, vous pouvez par exemple regarder [cette vidéo du Joueur du Grenier](https://www.youtube.com/watch?v=FgT-ja_U3tY) (35 minutes).

En théorie d'ICC, vous avez découvert un certain nombre d'algorithmes bien connus de l'informatique.
L'un d'entre eux est l'algorithme de plus court chemin dans un graphe, dû à Edsger Dijkstra.
On peut supposer qu'un tel algorithme va permettre à un blob de choisir un chemin jusqu'à sa destination.

Ce n'est pas si simple, cependant, car l'algorithme de Dijkstra travaille sur un graphe *discret*, avec des nœuds (*nodes*) et des arêtes (*edges*).
Or, le plan de notre jeu est plutôt une approximation d'un univers continu, avec des pixels et mêmes des positions en flottants.
On ne peut pas supposer des positions entières en pixels, car cela ne nous permettrait pas de se déplacer en diagonale comme de manière cardinale avec une norme constante (vous avez peut-être remarqué que votre joueur se déplace plus vite en diagonale qu'horizontalement ou verticalement !).
Et même si on pouvait faire ça, cela créerait un graphe trop dense, demandant trop de temps de calculs.

La solution est de *discrétiser* le plan avec un graphe abstrait.
Les nœuds du graphe représentent les points de passage possibles pour les chemins.
On connecte avec des arêtes les nœuds qui sont "à côté" sur le plan.
Ce graphe s'appelle un *navigation mesh*, ou *navmesh* en abbrégé.

Voici une image du jeu où nous avons dessiné le navmesh, et où nous avons mis en évidence le chemin "prévu" d'un blob :

![Navigation mesh](/assets/img/projet/navmesh.png)

Ici, on a choisi de créer un seul nœud par cellule de la map.
Le nœud est placé au centre de la cellule.
Important : on ne crée *pas* de nœud pour les cellules qui comportent un obstacle !
On a ensuite connecté les nœuds qui sont "à côté" les uns des autres, soit verticalement ou horizontalement, soit en diagonale.

On voit que le chemin calculé par le blob est le/un *plus court chemin* vers sa destination.
En particulier, on voit qu'il choisit de suivre une diagonale plutôt que deux côtés d'un carré.
Mais remarquez aussi qu'il "sait" qu'une diagonale est plus longue qu'un seul côté.
Si ce n'était pas le cas, il aurait pu faire des zigzags selon les diagonales dans la ligne droite à la fin de son chemin.
C'est parce que les arêtes du graphe ont un *poids* (*weight*), qui est la distance (euclidienne) entre les deux nœuds.
Un chemin qui prend deux diagonales a donc un poids total de $\sqrt{2} \cdot s$, alors qu'un chemin qui utilise deux côtés cardinaux a un poids de $s$ (où $s$ est la taille en pixels d'un côté de cellule).

Notez que le départ et l'arrivée sont des positions arbitraires dans le plan.
Dans le cas général, ces positions ne font *pas* partie des nœuds !
Si on veut aller d'un point $S$ (source) à un point $T$ (target), il faut donc :

1. Trouver le point $S'$ le plus proche de $S$ parmi les nœuds du navmesh.
2. Trouver le point $T'$ le plus proche de $T$ parmi les nœuds du navmesh.
3. Calculer un plus court chemin $(S', \ldots, T')$ de $S'$ à $T'$ avec l'algorithme de Dijkstra.
4. Le chemin total est $S, S', \ldots, T', T$.

Finalement, on ira en ligne droite d'un des points du chemin jusqu'au suivant.

### Navmesh plus fin

Avec un seul nœud par cellule de la map, les blobs marcheront encore "un peu" sur les buissons lorsqu'ils passent en diagonale à côté de ceux-ci.
On peut régler ceci en augmentant la densité du navmesh, c'est-à-dire en créant plusieurs nœuds par cellule.

On peut par exemple utiliser $3 \times 3 = 9$ nœuds par cellule.
1 au centre (comme avant), et les autres placés régulièrement autour.
Sur la largeur, pour $s$ la taille en largeur d'une cellule (en pixels par frame, donc ce que nous avons appelé `TILE_SIZE`), on aura des nœuds à $s/6$, $3s/6$ et $5s/6$ par rapport au bord de la cellule ($3s/6 = s/2$ étant au centre).
Le nœud le plus à gauche de la cellule d'à côté aura donc un nœud à $s + s/6 = 5s/6 + 2s/6$, et donc le grillage sera régulier.

Pour garder un grillage régulier, et pour que le centre de la cellule fasse toujours partie du navmesh, on utilisera toujours $n^2$ nœuds par cellule avec $n$ un entier impair.
1 nœud, 9 nœuds, 25 nœuds, etc.
De manière générale pour $n \times n$ nœuds par cellule, leurs positions se situeront à $(2i+1)s/(2n)$ pour $i \in [0, n)$.

⚠️ Attention cependant !
On a maintenant des nœuds qui, n'étant pas au centre d'une case non-obstacle, seront trop près d'un buisson.
On évitera de créer des nœuds se retrouvant à une distance de moins de $s$ du centre d'un *buisson*.
Par contre, on a parfaitement le droit de créer des nœuds proche des *trous*.

### Ligne de vue sur le joueur

Lorsqu'un blob "voit" le joueur, à n'importe quelle frame, il abandonne sa destination courante et la remplace par la position courante du joueur.
Un blob voit le joueur s'il y a une *ligne de vue* depuis son centre jusqu'au centre du joueur.

Utilisez la fonction [`arcade.has_line_of_sight`](https://api.arcade.academy/en/stable/example_code/line_of_sight.html).
Les buissons sont les seuls "murs" qui bloquent la vue.

Vous voudrez aussi probablement limiter la vue à une distance maximale.
Par exemple, si la distance entre le blob et le joueur est de plus de $5s$, vous pourriez considérer qu'il est trop loin pour être vu, qu'il y ait des buissons ou pas.

En particulier, les *trous* ne bloquent pas la vue, mais sont des obstacles.
Un blob peut donc repérer le joueur par-dessus un trou, mais devoir faire un détour pour le pourchasser !
Veillez à tester ce scénario, qui est tout l'intérêt de cette histoire.

### NetworkX

Pour représenter le navmesh et calculer les plus courts chemins, utilisez la bibliothèque [NetworkX](https://networkx.org/documentation/stable/tutorial.html).
Vous aurez besoin d'une bibliothèque de stubs de types pour que ty la comprenne à peu près.
Installez donc

```bash
$ uv add networkx types-networkx
```

Suivez le tutoriel (lien ci-dessus) pour apprendre les bases de cette bibliothèque.
Le tutoriel vous proposera du code Python *non typé* !
Lorsque vous créez un graphe, typez-le explicitement comme suit :

```python
graphe: nx.Graph[TypeNoeud] = nx.Graph()
```

où `TypeNoeud` est le type que vous choisirez pour représenter les nœuds du graphe.
Comme l'indique la documentation, ce type doit être [hashable](/tutoriels/structures-de-donnees.html#la-méthode-spéciale-__hash__).

En plus des bases, vous aurez besoin d'[une des fonctions implémentant l'algorithme de Dijkstra](https://networkx.org/documentation/stable/reference/algorithms/shortest_paths/dijkstra.html).
Ne sous-estimez pas la notion de "weight".

### Proposition de plan d'implémentation

Nous vous proposons les étapes suivantes pour votre implémentation, afin de diviser le travail dans le temps et de pouvoir avancer de manière concrète par petits bouts :

1. Ajoutez les blobs, avec leurs *destinations aléatoires possibles*, mais sans navmesh.
   Quand il a choisit sa destination courante, le blob va en ligne droite depuis sa position actuelle jusqu'à sa destination.
   Oui, il va donc traverser les obstacles à ce stade.
2. Ajoutez le navmesh, avec 1 nœud par cellule de la map, au centre de celle-ci.
   Suivez désormais ce navmesh plutôt qu'aller en ligne droite jusqu'à la destination.
3. Ajoutez la détection et "l'attaque" du joueur, sur base de la ligne de vue.
4. Affinez votre navmesh avec plusieurs nœuds par cellule.

**Questions de design :**

* Qu'avez-vous choisi comme type de nœud `TypeNoeud` ? Pourquoi ?
* À quel niveau traitez-vous la construction du navmesh (dans `Map`, dans votre map reader, dans la gestion des blobs, etc.), et où le stockez-vous ? Pourquoi ces choix ?
* Si vous avez $n \times n$ nœuds par cellule, et une carte de taille $m \times $m$, quelle est la complexité de vos différents algorithmes ?

### Afficher le navmesh pour déboguer

Cette section est entièrement optionnelle et est ici uniquement pour vous aider.

Si vous voulez dessiner le navmesh et les chemins qu'ont choisis vos blobs, comme sur l'image ci-dessus, vous pouvez ajouter le code suivant dans votre `GameView.on_draw` :

```python
DRAW_NAVMESHES: Final[bool] = False
"""Whether to draw navmeshes. Kills performance when True."""

...

    def on_draw(self) -> None:
        ...
        with self.camera.activate():
            ...

            if DRAW_NAVMESHES:
                nodes: list[TypeNoeud] = list(the_navmesh.nodes)
                for node in nodes:
                    pos: arcade.Vec2 = position_of_node(node)
                    arcade.draw_circle_filled(pos.x, pos.y, 2, arcade.color.BLACK)
                    for neighbor in the_navmesh.neighbors(node):
                        neighbor_pos: arcade.Vec2 = position_of_node(neighbor)
                        arcade.draw_line(pos.x, pos.y, neighbor_pos.x, neighbor_pos.y, arcade.color.BLACK, 2)

                for slime in ...: # iterate on your slimes
                    path: Sequence[arcade.Vec2] = somehow_get_path_of_slime(slime)
                    arcade.draw_line_strip(path, arcade.color.RED, 2)
```

Évidemment, il conviendra d'adapter avec vos fonctions pour `position_of_node` et `somehow_get_path_of_slime`, en fonction de comment vous aurez décidé de représenter vos nœuds et vos chemins.

## Interrupteurs et portails

Les interrupteurs ouvrent et ferment des portails lorsqu'ils sont touchés par une arme (épée ou boomerang).

Nous ajoutons deux nouveaux types de cases dans nos fichiers de map :

* un interrupteur `^` (*switch*)
* un portail `|` (*gate*)

### Interrupteurs

L'interrupteur a deux états : "on" et "off".
Par défaut, chaque interrupteur est dans l'état "off".
Vous aurez donc besoin de deux textures.
Vous pouvez utilisez les ressources suivantes, fournies par Arcade :

* `":resources:/images/tiles/leverLeft.png"` (off)
* `":resources:/images/tiles/leverRight.png"` (on)

Ces textures font 128x128 pixels.
Pour les mettre à la même taille que le reste de nos graphismes, vous devrez appliquer `scale=0.25` à leur sprites, et non `scale=2` comme d'habitude.

Les interrupteurs n'interagissent pas avec le joueur.
Il peut passer dessus comme s'ils n'existaient pas.
De même, ils ne sont pas des obtables pour les blobs, et n'entravent pas leur ligne de vue.

Lorsqu'un interrupteur est touché par une arme (épée ou boomerang, y compris au "retour" du boomerang), il change d'état.
S'il était on, il devient off, et inversement.

Le boomerang passe dans l'état "returning" s'il était dans l'état "launching", comme lorsqu'il touche un monstre.

### Portails

Un portail peut être ouvert (il laisse passer le joueur) ou fermé (il ne le laisse pas passer).
Chargez l'image `"assets/Top_Down_Adventure_Pack_v.1.0/Dungeon_Tileset.png"` comme une grille de taille 13x12 (comme pour `overworld_grid`) et utilisez les éléments $(4, 8)$ (ouvert) et $(7, 8)$ (fermé).

> Pour être vraiment joli, un portail fermé devrait en fait couvrir 2 cellules (donc 2 sprites), où celle du haut utilise l'élément $(6, 8)$ de la grille.
> Ce sera cependant nettement plus difficile à traiter dans votre code.
> On ne vous le demande donc pas.

Le portail, quand il est fermé, est un mur, au sens où il ne permet pas au joueur de passer.
Lorsqu'un portail est ouvert, le joueur peut passer.

Les portails, même ouverts, sont des *obstacles* pour les blobs (on fait ça pour le navmesh ne change pas pendant le jeu).
En revanche, seuls les portails *fermés* bloquent leur ligne de vue.

La grande question est : quand sont-ils ouverts ou fermés ?
On va configurer, dans notre fichier de map, les conditions sous lesquelles chaque portail est ouvert ou fermé.

On va donc devoir étendre la section de configuration de notre fichier de map.
Jusqu'à présent, elle contenait toujours exactement deux informations : `width` et `height`.
On va maintenant ajouter deux nouvelles sections : `switches` et `gates`.

Voici un exemple de map qui possède deux interrupteurs et deux portails.

```yaml
width: 40
height: 15
switches:
  - id: first
    x: 1
    y: 7
    # this switch is off when the game starts (no 'state')
  - id: second
    x: 8
    y: 6
    state: on # this switch is on when the game starts
gates:
  - x: 5
    y: 7
    # open the gate iff the switch 'first' is on *and* the switch 'second' is *not* on
    open_if:
      and:
        - switch_is_on: first
        - not:
          - switch_is_on: second
  - x: 10
    y: 3
    # open the gate iff the switch 'first' is off *or* the switch 'second' is off
    open_if:
      or:
        - not:
          - switch_is_on: first
        - not:
          - switch_is_on: second
---
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
x  *               S     x             x
x                        x    *   *    x
x                      * x             x
x         s        xxxxxxx            sx
x     xxx *   xx                       x
x                            xxx       x
x^   |  v    xxxxxxxxxx        x       x
x       ^ *                    x   *   x
x      S              S     *  x       x
x  *       v    *              O       x
x         |                    O       x
x     xx     s        xs               x
x  P      *           x       *        x
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
---
```

Le format général dans lequel est écrite cette configuration est du [YAML](https://yaml.org/).
YAML est un format similaire à JSON (dont on parle [dans la série 8](/series/08-patmat-et-bibliotheques.html)) : il peut représenter des valeurs primitives (nombres, chaînes de caractères, booléens), des listes et des dictionnaires.
Cependant, là où JSON est optimisé pour les machines, YAML est plutôt prévu pour être *écrit* (et relu) par des humains, puis interprété par des machines.
Les lignes qui commencent par `#` sont des *commentaires* en YAML (comme en Python) ; ces lignes peuvent donc bien faire partie du fichier, tel que montré ci-dessus.

YAML n'est pas un format facile à lire, pour un programme.
On ne s'attend pas à ce que vous puissiez écrire vous-même un algorithme pour "parser" correctement du YAML.

Vous devrez donc utiliser une bibliothèque adéquate pour lire tout ça.
Par "chance", nos quelques configurations existantes (`width` et `height`) s'avèrent déjà être du YAML valide.
On peut donc les intégrer au nouveau système.

Conseil : commencez justement par adapter votre lecture de la configuration *existante* en tant que YAML, avant d'ajouter les interrupteurs et les portails.

Il existe plusieurs bibliothèques disponibles pour lire du YAML en Python.
À vous de faire preuve de recherche, d'esprit critique et d'expérimentation pour choisir celle qui vous convient le mieux.

La configuration `switches` est une *liste*, dont chaque élément est un dictionnaire avec 4 clefs.
Les clefs `x` et `y` indiquent la position du switch (en coordonnées de grille) ; il doit donc y avoir un `^` à cet endroit de la map.
La clef `id` lui donne un nom (une `str`) pour pouvoir lui faire référence depuis les conditions des portails.
La clef `state` est soit `off` soit `on` et indique l'état dans lequel le switch commence le jeu.
Cette dernière clef est optionnelle ; si elle n'est pas présente, elle est `off`.

La configuration `gates` est plus compliquée.
Là aussi, c'est une liste où chaque élément est un dictionnaire.
Les clefs `x` et `y` indiquent la position du portail (où il doit y avoir un `|`).
La clef `open_if` décrit les conditions sous lesquelles il est ouvert.

Il s'agit d'une formule logique, définie sous forme *récursive*.
Une formule logique $P$ est un dictionnaire avec une seule clef, qui peut être soit :

* `and`, où la valeur associée est une liste de 2 éléments $Q_1$ et $Q_2$, tous deux récursivement des formules logiques ;
* `or`, avec la même structure ;
* `not`, où la valeur est une liste de 1 élément $Q$ qui est récursivement une formule logique ;
* `switch_is_on`, où la valeur est une `str` qui doit est l'`id` d'un des switches dans la la configuration `switches`.

Il peut donc y avoir un `and` dans un `not`, lui-même dans un `or` !
Il faut gérer les formules arbitrairement complexes.

Ici, on a encodé la formule $\text{first} \land (\lnot \text{second})$.
À tout moment (mis à jour à chaque frame), ce portail est donc ouvert si et seulement si le switch `first` est on et le switch `second` est off.

Si la configuration ne respecte pas le format, pour une raison ou une autre, le chargement de la carte devra lancer une `InvalidMapFileException`.

**Questions de design :**

* Quelle structure de données utilisez-vous pour représenter les conditions d'ouverture des portails ? Pourquoi ?
* Pouvez-vous tester l'évaluation des formules logiques sans dépendre de Arcade ?
* S'il y a $n$ interrupteurs et $m$ portails, et en supposant que chaque condition de portail n'est qu'un unique `switch_is_on` (pas de `and`, `or`, `not`), quelle est la complexité de traitement des portails à chaque frame ?

## Compléter le log

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).
