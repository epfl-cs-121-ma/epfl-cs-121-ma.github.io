---
title: Épée et chauves-souris
layout: default
use_mathjax: true
---

# Épée et chauves-souris

Cette semaine, nous complexifions deux aspects de notre jeu.
Nous ajoutons une nouvelle arme, l'épée, ainsi qu'un nouveau monstre, la chauve-souris.

Répartissez-vous les tâches de manière à pouvoir travailler indépendemment et en parallèle.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Épée

Pour l'instant, votre joueur n'a qu'un boomerang à sa disposition.
Nous allons lui ajouter une épée.
Lorsque le joueur appuie sur la touche R du clavier, il *change d'arme active*.
Par défaut, l'arme active est le boomerang.
Lors de l'appui sur la touche D, c'est l'arme active qui est utilisée, dans la direction actuelle du joueur.

Ajoutez une icône en haut à gauche de l'écran qui montre en tous temps quelle est l'arme active.

L'animation d'attaque à l'épée dépend de la *direction* actuelle du joueur, en suivant le modèle `"assets/Top_Down_Adventure_Pack_v.1.0/Char_Sprites/char_attack48_{dir}_anim_strip_6.png"`.
Ces animations comportent 6 frames de 50 ms chacune.
Attention, les frames ont une taille de 48x48 pixels et non les 16x16 habituels.
Utilisez le paramètre `tile_size` de `_load_animation_strip` pour charger la bonne taille.

Si l'épée est active lors de l'appui sur D, les événéments suivants doivent se produire :

1. Un sprite d'attaque apparaît, choisi avec le bonne direction, centré sur la position actuelle du joueur.
   L'épée devient alors *active*.
2. Son animation commence au temps 0 au moment où elle apparaît.
3. L'épée reste active tant que l'animation est en cours.
   Pendant ce temps, l'appui sur la touche D ne fait rien.
   Si l'épée entre en collision avec un monstre, celui-ci est vaincu.
   De plus, si elle entre en collision avec un **crystal**, le joueur collecte ce crystal (ceci n'était pas vrai avec le boomerang).
4. Une fois l'animation terminée (après 6x50 ms), l'épée redevient *inactive*.
   Elle disparaît, et la touche D reprend du service.

*Optionnel* : pendant que l'épée est active, bloquez les déplacements du joueur et cachez son sprite.
Vous aurez alors un meilleur rendu graphique.

*Optionnel* : pendant qu'une arme est active (épée ou boomerang), bloquez l'effet de la touche R aussi.

L'épée n'interagit pas avec les autres sprites sur le jeu (comme les buissons).
Elle passe à travers.

Si un monstre attaque le joueur "par derrière" pendant que l'épée est active, que devrait-il se passer ?
Est-ce que votre implémentation a le comportement attendu ?

**Question de design** : Comment gérez-vous le fait que vous avez maintenant deux types d'armes, avec des comportements différents ?
Pensez-vous que vous pourriez ajouter une troisième arme sans tout refaire ?

## Chauves-souris

Les chauves-souris sont un nouveau type de monstre, pour déjouer les plans du joueur.

Comme les spinners, si le joueur touche une chauve-souris, il perd.
De même, si l'épée ou le boomerange touche une chauve-souris, celle-ci est tuée et retirée du jeu.

L'animation pour la chauve-souris est disponible dans `"assets/Top_Down_Adventure_Pack_v.1.0/Enemies_Sprites/Pinkbat_Sprites/pinkbat_idle_left_anim_strip_5.png"` (5 frames).

On place une chauve-souris sur la carte avec un nouveau caractère : `v`.
Sa position de départ définit un "champ d'action" dans laquelle elle est autorisée à voler.
À vous de définir ce champ d'action : cela peut être un disque centré sur la position de départ, ou un rectangle, etc.
Le champ d'action de chaque chauve-souris est défini au chargement de la carte, et ne change plus après.

Le *comportement* de la chauve-souris est erratique et aléatoire.
Mais on ne veut pas non plus qu'elle soit *trop* aléatoire.
Il faut faire en sorte que son mouvement ait l'air plus ou moins naturel.

Voici d'abord quelques règles strictes à respecter :

* Les chauves-souris ne sont affectées par *aucun* type de sprite (à part les armes qui les tuent).
  Elles volent "à travers" les buissons, les crystaux, les autres monstres, etc.
* Elles ne peuvent pas sortir de leur champ d'action.
* Elles doivent avoir une vitesse de norme constante (mais dont la direction variera).
* Leur trajectoire doit avoir une part de hasard ; elle ne peut pas être entièrement déterministe.

Nous ne prescrivons pas de formule précise pour leurs mouvements.
Voici en revanche quelques "idées" qui peuvent vous lancer :

1. Commencez par la faire se déplacer en ligne droite, à vitesse constante.
   Si elle s'apprête à sortir de son champ d'action, changez de direction.
   Soit aléatoirement, soit en "rebondissant" sur la paroi invisible du champ d'action, soit en faisant demi-tour, etc.
2. Ensuite, toutes les "quelques" frames, donnez-lui l'opportunité de changer de direction *aléatoirement*.
3. Finalement, faites en sorte que cet "aléatoirement" ait plus de chances de rester plus ou moins dans la même direction qu'avant, jusqu'à avoir très peu de chances de faire un demi-tour complet.
   Autrement dit, votre tirage aléatoire ne doit plus être *uniforme*, mais être *biaisé*, d'une manière ou d'une autre, autour de la direction actuelle de la chauve-souris.

Pour générer des nombres aléatoires en Python, consultez la [documentation sur les distributions à valeurs réelles du module `random`](https://docs.python.org/3/library/random.html#real-valued-distributions).

Pour vos tests, vous voudrez peut-être que le comportement des chauves-souris reste exactement le même d'une fois à l'autre (tout en suivant l'algorithme "aléatoire" que vous avez écrit).
Pour cela, vous pouvez créer une instance de `random.Random(seed)` où `seed` est une constante entière, toujours la même.
Pour le jeu normal (hors tests), utilisez une instance créée avec `None` comme seed (afin que chaque nouvelle partie soit différente).

**Question de design** : Comment gérez-vous le fait que vous avez maintenant deux types de monstres, avec des comportements différents ?
Pensez-vous que vous pourriez ajouter un troisième monstre sans tout refaire ?

## Compléter le log

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).
