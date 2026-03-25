---
title: Refactoring 1
layout: default
use_mathjax: true
---

# Refactoring 1

Cette semaine, nous n'introduisons aucune nouvelle fonctionnalité dans votre projet.
Elle est dédiée à :

* vérifier les aspects "administratifs" du projet ;
* rattraper le retard éventuel ;
* y compris écrire les tests que vous auriez laissés de côté jusqu'ici ;
* *refactoriser* votre code pour améliorer sa qualité, le simplifier, et mieux le préparer aux évolutions futures.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Vérifier les aspects administratifs

Au cours des dernières semaines, j'ai remarqué qu'un certain nombre de projets ne respectent pas tous les aspects administratifs.
Vérifiez donc bien toutes et tous les aspects suivants.

### `uv` est la source de vérité

Que vous ayez déjà installé Python sur votre machine avant ce cours ou non ; que vous utilisiez VS Code ou PyCharm ou non ; la *source de vérité* reste `uv`.
Si VS Code ne montre pas d'erreur, mais que `uv` oui (ou inversement), **c'est `uv` qui a raison**.

Il est **impératif** que les cinq commandes suivantes s'exécutent avec succès dans votre projet :

* `uv run ruff check` : Ruff doit écrire "All checks passed!" ;
* `uv run ty check` : ty doit écrire "All checks passed!" (sans avoir recours à aucun `# type: ignore`, voir plus bas) ;
* `uv run pytest` : tous vos tests doivent s'exécuter avec succès ;
* `uv run main.py` : la seule commande que nous utiliserons pour lancer votre projet avec votre map par défaut ;
* `uv run main.py chemin/vers/une_autre_map.txt` : la commande que nous utiliserons pour tester votre projet avec nos maps.

Il est **inacceptable** de rendre un projet pour lequel une ou plusieurs de ces commandes échouent.
Une **forte pénalité** sera appliquée à tout projet qui ne respecterait pas ces règles.

---

De manière générale, je trouve que beaucoup d'entre vous ont "normalisé" la présence d'erreurs de types dans votre code.
Ce n'est pas une attitude saine.

Préservez en tous temps un projet sans erreur de types.
Bien sûr, pendant que l'on écrit, des erreurs transitoires vont apparaître ; mais il devrait être rare de vouloir même *lancer le programme* alors qu'il subsiste une erreur de type quelque part.

Cela reste possible, si on a la conviction que le code est correct et qu'on n'a pas encore réussi à convaincre ty que c'est le cas.
Il est alors possible de lancer le programme et les tests pour s'en assurer.
Si ça marche, alors on se remet immédiatement à la tâche de convaincre ty, avant de faire quoi que ce soit d'autre.
Et si ça ne marche pas, il est probable que ty avait raison !

### Ruff doit être bien configuré

Il est **impératif** que Ruff soit configuré pour rendre les annotations de types obligatoires.
Vous devez avoir les deux lignes suivantes dans votre `pyprojet.toml`, telles que données sur la [référence de création d'un projet](/references/quick-projet-setup.html) :

```toml
[tool.ruff.lint]
select = ["ANN"]
```

### Aucun `# type: ignore`

Sauf cas exceptionnel *personnellement autorisé par l'enseignant* (et lui seul), il **est interdit** d'insérer un

```python
# type: ignore
```

dans votre code, ni tout autre forme de commentaire qui ferait un bypass des règles de ty.

Cette indication revient à *annuler* les vérifications de ty, et fait donc partie des pénalités mentionnées plus haut.

## Rattraper le retard éventuel

Votre principal objectif pour cette semaine est de rattraper le retard que vous auriez éventuellement accumulé au cours des semaines précédentes.

En particulier, s'il y a des fonctionnalités pour lesquelles vous n'avez encore aucun scénario de test, c'est le moment d'y remédier.
Vous devez avoir au moins un scénario de test pour chaque fonctionnalité demandée.

(Pour rappel, nous ne nous attendons pas à ce que vous testiez les effets sonores, car ce n'est pas possible.)

## Refactoring

Votre second objectif est de profiter de cette semaine pour faire du *refactoring*.
Le refactoring, c'est quand on améliore la qualité du code sans changer les fonctionnalités.
Corriger un bug n'est donc pas considéré comme du refactoring (même si, parfois, un refactoring bien conçu peut corriger un bug par construction).

Au cours des dernières semaines, vous avez probablement ajouté frénétiquement les fonctionnalités demandées, sans prendre (beaucoup) de temps pour réfléchir à la qualité de votre code.
Cette semaine est donc dédiée à s'arrêter, à réfléchir, et à refactoriser.

Le but des refactoring est de mieux préparer une codebase aux évolutions futures.

**Avant** de faire du refactoring, on s'assure que nos tests sont satisfaisants.
Bien que bénéfique pour la suite, chaque refactoring comporte une part de danger immédiat (comme les vaccins).
Avoir de bons tests minimise les risques qu'un refactoring casse accidentellement quelque chose qui fonctionnait.

Voici quelques exemples de refactoring qui pourraient vous tenter :

* Décomposer une méthode qui est devenue ingérable en sous-méthodes.
* Extraire un algorithme compliqué pour le rendre indépendant d'Arcade (sauf peut-être `arcade.Vec2`), ce qui permet de le *tester plus facilement*.
* Identifier les endroits où il y a beaucoup de code "compliqué" pour une fonctionnalité, et réévaluer votre approche.
  Peut-être y a-t-il un moyen plus simple de s'attaquer au problème ?
  Exemple probable, d'après ce que j'ai vu dans beaucoup de vos projets : la gestion des 4 directions du joueur.
* Identifier le code dupliqué, et introduire de nouvelles méthodes et/ou classes pour y remédier.
* Introduire de nouvelles classes pour concentrer les différents traitements associés à une fonctionnalité au même endroit (plutôt qu'être disséminés à travers la classe `GameView`, par exemple).
* Identifier les problématiques qui pourraient être mieux traitées avec les concepts vus au cours : polymorphisme, héritage, structures de données, génériques, etc.
* Redistribuer vos classes dans différents *modules* (fichiers).
  Si presque tout votre jeu est encore dans `gameview.py`, ce fichier doit commencer à vous faire peur par sa taille.

Si vous avez l'impression que votre code est déjà superbement organisé, appelez-moi pendant la séance de jeudi, et je vous garantis qu'on trouvera quelque chose à améliorer.

## Questions fréquentes

Voici des réponses à quelques questions qui sont revenues fréquemment à propos du projet.

### Avons-nous le droit de ... ?

**Oui.**

Vos *devoirs* sont :

* respecter les aspects administratifs rappelés ci-dessus ;
* implémenter les fonctionnalités qui vous sont demandées dans les instructions du projet, les tester et les documenter.

Tant que vous remplissez vos devoirs, vous avez le *droit* de faire tout ce que vous voulez dans votre projet.
Quelques exemples (non exhaustifs) de ce dont vous avez le droit :

* ajouter des fonctionnalités qui vous plaisent ;
* changer tout ce qui est aspect graphique ;
* ajouter des bibliothèques supplémentaires (en plus d'Arcade) pour vous faciliter la vie.

### Sur quoi porte l'évaluation ?

Par ordre *décroissant* d'importance :

* L'usage adéquat des concepts vus au cours ;
* La qualité de votre code, y compris
  * son intelligibilité ;
  * sa documentation : commentaires et fichier `DESIGN.md` ;
* Les tests `pytest`, couvrant au moins un scénario par fonctionnalité qui vous a été demandée ;
* Les fonctionnalités qui vous sont demandées au fil des instructions du projet ;
* (Analyse de performance ; mais ceci viendra plus tard dans le cours et le projet).

Oui, vous avez bien lu.
Il est plus important d'avoir un bon design et de bien le documenter que d'avoir chaque fonctionnalité parfaitement en adéquation avec les instructions.
Ceci est cohérent avec la vie réelle : il est plus facile de corriger un bug dans un projet bien conçu.

À propos du premier point, il est tout à fait possible --- probable, même --- que votre projet n'ait pas besoin de *tous* les concepts vus au cours.
En fonction de votre design, différents concepts seront appelés à être mobilisés.
Ce que l'on évalue, c'est que, *là où votre problématique appelait un concept*, que votre solution exploite adéquatement ce concept.
Par exemple, si vous êtes devant une problématique où différentes "choses" doivent être traitées de manière uniforme tout en ayant des comportements distincts, et que vous passez totalement à côté du *polymorphisme* comme une solution à cette problématique, vous allez perdre des points.

Je cite le polymorphisme en particulier car c'est précisément le chapitre du cours que nous avons introduit comme

> sans doute le point le plus important de tout notre cours.

Si à ce stade du projet, vous n'avez introduit de polymorphisme nulle part, vous êtes presque certainement passé-e à côté de quelque chose d'important.

Voici deux autres exemples de "ratés" possibles, par rapport à des concepts que vous avez déjà vus :

* vous avez écrit un algorithme en $\Theta(n^2)$ alors qu'il est aurait pu être $\Theta(n)$ avec l'usage adéquat d'un [`set` ou d'un `dict`](/tutoriels/structures-de-donnees.html) ;
* vous avez recopié deux fois la même classe/fonction sur des types différents, alors qu'un peu de [polymorphisme paramétrique](/tutoriels/generiques-et-lambdas.html) aurait permis de n'en écrire qu'une.

#### *Quid* de nos extensions ?

À la fin du projet, vous devrez ajouter 1 ou 2 extensions, **validées par l'enseignant au préalable**, qui vous seront propres et seront évaluées.
La validation est nécessaire pour vérifier que la difficulté est adéquate (ni trop ni trop peu) et que les extensions permettent d'illustrer les concepts vus au cours.

**En dehors de cela**, l'évaluation **ne porte pas** sur des fonctionnalités ou aspects graphiques que vous auriez implémentés en plus de ce qui vous est demandé par les instructions du projets.
Les aspects sonores ne sont pas évalués.
D'autres extensions non validées par l'enseignant, quelles qu'elles soient, ne rapporteront pas de points, même en bonus.
La qualité de gameplay (est-ce que le *jeu* est fun à jouer ou pas) non plus.
Tout au plus, ces aspects pourront "mettre de bonne humeur" votre correctrice ou correcteur, ce qui n'est pas perdu.

Gardez toutefois à l'esprit que la qualité de votre code et sa documentation sont évalués sur *l'ensemble de votre projet*.
Si votre code est tout pourri à cause d'extensions que vous avez choisi d'ajouter, c'est quand même du code tout pourri, qui sera sanctionné comme tel.
Cela va dans l'autre sens également : si vous avez trouvé des moyens élégants de concevoir votre application pour tenir compte de vos extensions, cela est valorisé.

Je vous mets particulièrement en garde contre la tentation de créer vous-mêmes vos propres images et sons.
C'est un métier à part, et une activité chronophage si on ne maîtrise pas cet art.
Si vous y tenez quand même, sachez que vous le faites comme un *hobby* qui ne sera en aucun cas pris en compte dans l'évaluation de votre travail.

### C'est vraiment important, les tests pytest ?

**Oui.**

Écrire de tests unitaires (tels qu'avec pytest) est une partie intégrante de la discipline d'ingénieur-e en informatique.
Sans ça, on peut se qualifier de *script kiddies*, mais pas d'ingénieur-e.

On ne fait pas de mathématiques sans preuves ; on ne fait pas de physique des matériaux sans tests de résistance ; on ne fait pas d'électronique sans tests aléatoires des batches produits.
De la même façon, on n'écrit pas de logiciels sans les tests qui vont avec.

Si une fonctionnalité n'est pas testée dans un programme, il faut partir du principe qu'elle ne fonctionne pas.

> Mais tester "à la main" est tellement plus rapide !

C'est plus rapide *une fois*.
L'intérêt des tests unitaires n'est pas de tester *maintenant*, une fois, ce que vous êtes en train d'implémenter.
Leur intérêt est acquis dans le temps : ils servent à s'assurer que ce que vous implémentez aujourd'hui *continuera à fonctionner dans le futur*.

Les ajouts et modifications futures de votre programme vont, inévitablement, casser des aspects de votre programme qui fonctionnaient avant.
Les tests unitaires vous permettent de détecter ces situations et d'y remédier immédiatement.

Il est donc totalement contre-productif d'attendre la fin du projet pour ajouter des tests afin de remplir les conditions d'évaluations.
Vous passeriez à côté de leurs bénéfices (vous empêcher de casser des trucs qui fonctionnent), et n'en ressentiriez que les désavantages (le temps passé à les écrire).

**Toutefois, n'en faites pas trop !**

On vous demande *1 scénario de test par fonctionnalité* (y compris vos extensions).
Dans certains cas, vous jugerez qu'une fonctionnalité a besoin d'un peu plus de tests pour être bien couverte.
Mais il n'est pas nécessaire de tester 10 manières différentes dont le joueur pourrait tomber un trou.

> Dans la vraie vie, pour un programme bien testé, il devrait y avoir à peu près *autant de code de test* que de code d'implémentation.
>
> Par exemple, dans le projet principal que je mène, j'ai compté en 2025 environ 270 000 lignes de code en tout, dont au moins 114 000 lignes de tests (approximation par le bas de ce qui compte comme "tests").
> Les tests représentent donc au moins 42 % du projet.
>
> On ne vous en demande pas tant pour ce projet.

### Si nous avons un fichier de `DESIGN`, avons-nous aussi besoin de commentaires ?

**Oui.**

Le fichier `DESIGN` et les commentaires sont tous deux des éléments de *documentation technique* de votre projet.
Cependant, ils ne se remplacent pas les uns les autres.
Ils sont *complémentaires*.

Le fichier `DESIGN` présente la *structure générale* de votre projet.
Quelles sont les relations entre les principales classes ?
Quel design utilisez-vous pour répondre à telle ou telle problématique ?

Les commentaires sont une documentation *localisée*.
À quoi sert *cette* méthode ou *cette* classe ?
Comment cette méthode compliquée réalise son objectif ?

**Là aussi, inutile d'en faire trop !**
Il n'est pas nécessaire de commenter *chaque* méthode de votre projet (par contre, chaque classe mérite probablement son commentaire, même s'il est très succinct).
En fait, si vous avez fait ça, je serai pratiquement certain que vous avez juste demandé à une IA de le faire pour vous, ce qui ne m'intéresse pas.

Identifiez les méthodes qui en ont le plus besoin :

1. Les méthodes polymorphes sont généralement celles qui ont un besoin critique de documentation précise.
   En effet, il est impossible de respecter le LSP au travers du polymorphisme si la méthode n'a pas un contrat clair au départ.
2. Les méthodes qui sont le point d'entrée d'une fonctionnalité importante de votre programme, par exemple s'il y a un algorithme un peu complexe derrière.
   Un bon exemple à ce stade seraient la ou les méthodes qui calculent les bornes de déplacements de vos spinners et chauves-souris.
3. Toute autre méthode dont le nom et la liste des paramètres ne suffiraient pas à savoir ce qu'elle est censée faire.
   Souvent, dans ce dernier cas, une phrase ou deux sont suffisantes.
