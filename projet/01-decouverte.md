---
title: Découverte d'Arcade
layout: default
use_mathjax: true
---

# Découverte d'Arcade

Pour cette première semaine, nous allons découvrir la bibliothèque Arcade et ses principaux concepts.
Cette première série de tâches est fortement guidée, et peut difficilement être décomposée en parallèle.
Si vous avez déjà votre binôme, je vous conseille de :

* Développer ensemble sur une machine jusqu'au point indiqué (ou en passant d'une machine à l'autre, mais pas en même temps).
* Changer de personne qui *écrit* effectivement le code à chaque section.
* L'autre personne peut *lire* plus en détail ce document.

Si vous n'avez *pas* encore de binôme, cette semaine est encore une période de grâce.
Il est parfaitement possible de la compléter individuellement, en attendant de trouver votre binôme.
Vous pourrez alors partir de l'état de l'un ou de l'autre de vos projets respectifs.

{::options toc_levels="2..3" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Création du projet, affichage d'un écran bleu

### Création

Commencez par [créer un nouveau projet](/references/quick-projet-setup.html) pour votre... projet.

### Installation de Arcade

**tl;dr**: `$ uv add arcade`

Nous allons avoir besoin de la bibliothèque [Arcade](https://api.arcade.academy/en/latest/).
Il nous faut donc l'installer *dans notre projet*.
Heureusement, avec uv, cette tâche va se résumer à une ligne.

Comme avec toute bibliothèque, nous devons identifier quel est son **nom de paquet** (*package*).
La plupart des bibliothèques documentent comment les installer avec `pip`, mais pas avec `uv`.
Cependant, c'est le même nom de paquet dans les deux cas.
Nous allons donc à la section [Using pip](https://api.arcade.academy/en/latest/get_started/install.html#using-pip) de la page d'installation d'Arcade, qui nous dit :

> `pip install arcade` (ne pas exécuter ceci avec uv !)

Le nom du paquet est donc `arcade`.
Pour l'installer avec uv, nous exécutons donc :

```
$ uv add arcade
Resolved 18 packages in 162ms
Installed 9 packages in 31ms
 + arcade==3.3.3
 + attrs==25.4.0
 + cffi==2.0.0
 + pillow==11.3.0
 + pycparser==3.0
 + pyglet==2.1.13
 + pymunk==6.9.0
 + pytiled-parser==2.2.9
 + typing-extensions==4.15.0
```

Ça y est ! Arcade est installée.

Nous pouvons remarquer que uv a installé une série d'autres bibliothèques.
C'est parce qu'Arcade dépend elle-même de celles-ci.
uv installe automatiquement toutes nos dépendences *transitives*.
Par ailleurs, elles sont isolées dans ce projet.
Elles ne viennent pas encombrer d'autres projets sur votre ordinateur (voire entre en conflit avec *leurs* dépendences respectives).

### Ouvrir une fenêtre vide

Il va nous falloir un peu de "boilerplate" pour démarrer notre jeu.
Le boilerplate, c'est le code pas très intéressant qui fait "tourner la boutique".

Commençons par donner le code, puis nous allons expliquer ses concepts.

En plus du fichier `main.py`, créez deux autres fichiers `gameview.py` et `constants.py`.
Vous pouvez -- exceptionnellement -- copier-coller le code ci-dessous dans les fichiers correspondants.

`constants.py`

```python
WINDOW_TITLE = "Adventure"
"""Title of the main window."""

SCALE = 2.0
"""The global scale for all textures."""

TILE_SIZE = 32
"""After scaling, the size of a tile."""

MAX_WINDOW_WIDTH = 14 * TILE_SIZE
MAX_WINDOW_HEIGHT = 14 * TILE_SIZE
```

`gameview.py`

```python
from typing import Final
import arcade

from constants import *

class GameView(arcade.View):
    """Main in-game view."""

    world_width: Final[int]
    world_height: Final[int]

    def __init__(self) -> None:
        # Magical incantion: initialize the Arcade view
        super().__init__()

        # Choose a nice comfy background color
        self.background_color = arcade.csscolor.CORNFLOWER_BLUE

        # Setup our game
        self.world_width = 40 * TILE_SIZE
        self.world_height = 20 * TILE_SIZE

    def on_show_view(self) -> None:
        """Called automatically by 'window.show_view(game_view)' in main.py."""
        # When we show the view, adjust the window's size to our world size.
        # If the world size is smaller than the maximum window size, we should
        # limit the size of the window.
        self.window.width = min(MAX_WINDOW_WIDTH, self.world_width)
        self.window.height = min(MAX_WINDOW_HEIGHT, self.world_height)

    def on_draw(self) -> None:
        """Render the screen."""
        self.clear() # always start with self.clear()
```

```python
import arcade

from constants import *
from gameview import GameView

def main() -> None:
    # Create the (unique) Window, setup our GameView, and launch
    window = arcade.Window(MAX_WINDOW_WIDTH, MAX_WINDOW_HEIGHT, WINDOW_TITLE)
    game_view = GameView()
    window.show_view(game_view)
    arcade.run()

if __name__ == "__main__":
    main()
```

Vous pouvez immédiatement vérifier que tout se passe bien avec `uv run main.py`.

Détaillons maintenant ce que nous avons là, petit bout par petit bout.

`constants.py` définit quelques constantes que nous allons utiliser à plusieurs endroits dans le code.

Le fichier principal est en fait `gameview.py`.
On y importe le module `arcade` que nous fournit la bibliothèque Arcade, ainsi que les constantes que nous avons définies dans `constants.py` :

```python
import arcade

from constants import *
```

On définit notre classe principale, `GameView`.
On utilise une syntaxe particulière que nous n'avons pas encore vue, avec un qualificatif `(arcade.View)`.
Pour celles et ceux qui ont plus d'expérience : c'est la syntaxe qui nous permet *d'hériter de/étendre* `arcade.View`.
Pour les autres, acceptez pour l'instant que cette syntaxe est nécessaire pour qu'Arcade reconnaisse que notre classe représente notre jeu.

```python
class GameView(arcade.View):
    """Main in-game view."""
```

On définit deux attributs finaux représentant la taille logique du "monde" de notre jeu (la taille de la carte, si vous préférez).
Les longueurs géométriques seront toujours exprimées en pixels.
Comme ce sera vrai *partout* dans notre code, on ne le documente pas explicitement à chaque fois.

```python
    world_width: Final[int]
    world_height: Final[int]
```

Ils sont suivis du constructeur de notre classe.
Là aussi, une incantation magique, avec `super...`, se glisse dans le code.
Acceptez qu'elle est nécessaire en l'état, pour l'instant.

Après celle-ci, on initialise la couleur de fond d'écran.
Nous n'avons pas déclaré cet attribut.
C'est `arcade.View` qui l'a déclaré pour nous (oui, je sais, encore un peu de magie).

Finalement, on initialise nos attributs `world_width` et `world_height`.

```python
    def __init__(self) -> None:
        # Magical incantion: initialize the Arcade view
        super().__init__()

        # Choose a nice comfy background color
        self.background_color = arcade.csscolor.CORNFLOWER_BLUE

        # Setup our game
        self.world_width = 40 * TILE_SIZE
        self.world_height = 20 * TILE_SIZE
```

La méthode `on_show_view` calcule la taille que devra avoir la fenêtre, en fonction de la taille du monde et d'une taille maximale que nous avons définie dans `constants.py`.

```python
    def on_show_view(self) -> None:
        """Called automatically by 'window.show_view(game_view)' in main.py."""
        # When we show the view, adjust the window's size to our world size.
        # If the world size is smaller than the maximum window size, we should
        # limit the size of the window.
        self.window.width = min(MAX_WINDOW_WIDTH, self.world_width)
        self.window.height = min(MAX_WINDOW_HEIGHT, self.world_height)
```

Finalement, nous définissons la méthode `on_draw()`, qui est responsable de dessiner l'état du jeu à un instant donné.

```python
    def on_draw(self) -> None:
        """Render the screen."""
        self.clear() # always start with self.clear()
```

La méthode `clear()` nous est donnée par `arcade.View`.
Elle va remplir l'écran de la couleur `self.background_color`.

Notez que nous n'appelons pas `on_draw` nous-mêmes.
C'est Arcade qui va appeler cette méthode environ 60 fois pas secondes.
De même, `on_show_view` est appelée par Arcade.

Il ne nous reste plus que la fonction `main()`, qui est dans `main.py`.
Celle-ci crée une instance de la classe `arcade.Window`, qui représente la fenêtre de l'application.
Le constructeur attend les largeur, hauteur et titre de la fenêtre en paramètres.

Ensuite, nous créons une instance de notre classe `GameView`.
Nous passons cette instance en argument à la méthode `window.show_view()`, ce qui va installer notre vue "dans la fenêtre".
C'est grâce à cela que Arcade saura appeler nos méthodes `on_show_view` et `on_draw` !

Il ne reste plus qu'à lancer le "jeu" avec `arcade.run()`.

```python
def main() -> None:
    # Create the (unique) Window, setup our GameView, and launch
    window = arcade.Window(MAX_WINDOW_WIDTH, MAX_WINDOW_HEIGHT, WINDOW_TITLE)
    game_view = GameView()
    window.show_view(game_view)
    arcade.run()

if __name__ == "__main__":
    main()
```

Notez que nous ne connaissons pas l'implémentation de `arcade.Window`, pourtant nous pouvons l'utiliser correctement.
C'est le pouvoir des *interfaces* : nous n'avons pas besoin de connaître tous les détails de `arcade.Window`.

Il peut cependant être déroutant de comprendre ce code sans avoir une *idée* de la structure de `arcade.Window`.
Je vous propose donc, *pour une fois* une vue *schématisée, imaginée et grandement simplifiée* de ce que fait sans doute `arcade.Window` (je ne le sais pas moi-même, en fait !) :

```python
class Window:
    view: GameView

    def __init__(self, width: int, height: int, title: str) -> None:
        ... # toutes sortes d'initialisations

    def show_view(self, view: GameView) -> None:
        self.view = view
        view.on_show_view()

def arcade_run() -> None:
    window.open() # open the window on the user's screen
    while True:
        window.view.on_draw()
        wait_for(1 / 60) # wait for 1/60 seconds
```

Je vous suggère de créer un commit à ce moment.
De manière générale, il serait judicieux d'en faire un à chaque fin de section.

## Dessiner des objets : les Sprites

(Changer de membre du binôme qui "a le contrôle" ici.)

Il est temps de dessiner des objets à l'écran : notre personnage, de l'herbe et des buissons.
Dans un jeu, les objets ont un aspect *graphique* (dessin à l'écran) ainsi qu'un aspect *physique* (interactions).
Ceci est modélisé par la notion de *sprite*.
Chaque sprite représente un objet (ou parfois un morceau d'un objet) qui existe dans le monde 2D du jeu.

### Assets et Textures

Pour dessiner des images, il est utile d'avoir les images en question !
Avec les sons et autres contenus artistiques, on les appelle des *assets*.

Le sujet du cours n'étant absolument pas l'aspect artistique, nous vous fournissons un bundle contenant tout ce dont vous aurez besoin pour votre projet.
Téléchargez le fichier [`project-assets.zip`](./assets.zip).
Il contient un dossier `assets/`.
Extrayez le contenu de ce zip de sorte que le dossier `assets/` se retrouve à côté de votre fichier `main.py`.

La plupart des images fournies proviennent de [`o-lobster.itch.io`](https://o-lobster.itch.io/adventure-pack), où elles ont été mises à disposition sous licence [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/).
Cela nous permet de les réutiliser tant que nous en créditons la source.

Pour charger les images sous une forme exploitable par Arcade, on utilise une *texture*.
Il est recommendé de charger les images une seule fois de manière globale, plutôt qu'à chaque fois qu'on en a besoin.
Créez donc un nouveau fichier `textures.py` pour gérer les textures de votre jeu.

Le pack d'images que nous utilisons contient en fait des *animations* et des *spritesheets*.
Il va donc falloir extraire des sous-images disposées en grille dans de plus grandes images.
On vous donne donc encore un peu de boilerplate pour manipuler tout ça.

```python
from typing import Final
import arcade

ORIG_TILE_SIZE = (16, 16)

def _load_grid(
    file: str,
    columns: int,
    rows: int,
    tile_size: tuple[int, int] = ORIG_TILE_SIZE
) -> list[arcade.Texture]:
    """
    Loads a texture grid from a spritesheet.

    Args:
        file:
            Path to the spritesheet file name.
        columns:
            The number of columns in the grid.
        rows:
            The number of rows in the grid.
        tile_size (optional):
            The size in pixels of one element of the grid. Defaults to the
            standard tile size of `(16, 16)` that we use in our assets.

    Returns:
        A list of the loaded textures, flattened by row. The texture at grid
        coordinates `(x, y)` is at index `(y * columns) + x` in the list.
    """
    spritesheet = arcade.load_spritesheet(file)
    return spritesheet.get_texture_grid(tile_size, columns, columns * rows)

_overworld_grid = _load_grid("assets/Top_Down_Adventure_Pack_v.1.0/Overworld_Tileset.png", 18, 13)

TEXTURE_GRASS: Final[arcade.Texture] = _overworld_grid[18*1 + 6]
TEXTURE_BUSH: Final[arcade.Texture] = _overworld_grid[18*3 + 5]

TEXTURE_PLAYER_IDLE_DOWN: Final[arcade.Texture] = \
    _load_grid("assets/Top_Down_Adventure_Pack_v.1.0/Char_Sprites/char_idle_down_anim_strip_6.png", 1, 1)[0]
```

### Joueur

Commençons par dessiner notre joueur (*player*).
J'utiliserai le terme "joueur" en français car les graphismes que nous utilisons cette année représentent un homme.
Il faut bien alléger l'écriture à un moment.

Créons donc un sprite pour notre joueur.
Nous allons souvent manipuler celui-ci, donc nous allons le stocker dans un attribut de `GameView`.
Nous l'instancions dans `__init__()`.

À partir d'ici, je vous recommende de ne *pas* copier-coller les bouts de code.
Prenez la peine des les écrire vous-mêmes, afin d'entraîner votre mémoire par la mécanisation des opérations.
Vous pouvez bien sûr copier-coller les chaînes de caractères, surtout celles qui contiennent des chemins vers des ressources.

```python
...
from textures import *

def grid_to_pixels(i: int) -> int:
    return i * TILE_SIZE + (TILE_SIZE // 2)

class GameView(arcade.View):
    ...
    player: Final[arcade.Sprite]

    def __init__(self) -> None:
        ...
        self.player = arcade.Sprite(
            TEXTURE_PLAYER_IDLE_DOWN,
            scale=SCALE, center_x=grid_to_pixels(2), center_y=grid_to_pixels(2)
        )
```

Un sprite est toujours caractérisé par :

* une *texture*,
* une *échelle* (nous utiliserons toujours `SCALE`, que nous avons définie à `2.0` dans `constants.py`) et
* une *position* dans le monde.

Il ne nous reste plus qu'à *dessiner* notre joueur dans la méthode `on_draw` :

```python
    def on_draw(self) -> None:
        ...
        arcade.draw_sprite(self.player)
```

Si vous lancez le jeu à ce stade, vous devriez voir une image proche du coin inférieur gauche de l'écran.

### À propos du système de coordonnées

![Système de coordonnées de Arcade](/assets/img/projet/arcade-coordinate-system.svg){: class="floating-illustration-image"}

Arcade utilise un système de coordonnées correspondant à ce que vous connaissez des mathématiques.
Le point $(0, 0)$ est en bas à gauche de la fenêtre.
Les $x$ positifs vont vers la droite, et les $y$ positifs vont vers le haut.

C'est peut-être contre-intuitif si vous avez déjà programmé des systèmes graphiques dans d'autres environnements.
Souvent, en informatique, les $y$ positifs vont vers *le bas*.

Par contre, les *angles* de rotation sont traités par Arcade dans le sens des aiguilles d'une montre (*clockwise*).
Ceci est *l'opposé* de ce qu'on utilise généralement en mathématiques.

Gardez cela en tête lorsque vous vous battrez avec des calculs de coordonnées plus tard dans le projet.

### Collections de sprites avec `SpriteList`

Nous avons maintenant notre joueur, mais il lui manque un monde dans lequel évoluer.
Nous allons ajouter un bon nombre de sprites pour représenter de l'herbe et des buissons.

Nous pourrions les stocker dans une `list[arcade.Sprite]` de Python, puis faire une boucle `for` pour appeler `arcade.draw_sprite` sur chacun d'eux.
Cependant, cette approche poserait de sérieux soucis de performances.
À chaque appel à `draw_sprite`, Arcade doit interagir avec le GPU (Graphical Processing Unit) de votre ordinateur.
Ce va-et-vient est coûteux.
Il est beaucoup mieux de rassembler un grand nombre de sprites et de les envoyer tous ensemble au GPU.

C'est pourquoi Arcade a son propre type de collection, `arcade.SpriteList[T]`.
On peut appeler `draw()` directement sur une `SpriteList`, et cela envoie toute la collection d'un coup au GPU.
En fait, Arcade ne sait *que* utiliser `SpriteList`.
Quand on appelle `arcade.draw_sprite`, il crée une `SpriteList` temporaire pour l'envoyer au GPU !

On va donc déjà stocker notre joueur dans une `SpriteList` dédiée, et l'utiliser pour la dessiner.

> Ce qui suit est une vue de `git diff`.
> Les lignes qui commencent par `@@` indiquent qu'on "saute" une section de code qui ne change pas.
> Par exemple, la création de `self.player_list = arcade.SpriteList()` est bel et bien dans la méthode `__init__()`.
> Référez-vous au contexte donné.

```diff
diff --git a/gameview.py b/gameview.py
index daf6b3a..dda5f8b 100644
--- a/gameview.py
+++ b/gameview.py
@@ -14,6 +14,7 @@ class GameView(arcade.View):
     world_height: Final[int]

     player: Final[arcade.Sprite]
+    player_list: Final[arcade.SpriteList[arcade.Sprite]]

     def __init__(self) -> None:
         # Magical incantion: initialize the Arcade view
@@ -30,6 +31,8 @@ class GameView(arcade.View):
             TEXTURE_PLAYER_IDLE_DOWN,
             scale=SCALE, center_x=grid_to_pixels(2), center_y=grid_to_pixels(2)
         )
+        self.player_list = arcade.SpriteList()
+        self.player_list.append(self.player)

     def on_show_view(self) -> None:
         """Called automatically by 'window.show_view(game_view)' in main.py."""
@@ -43,4 +46,4 @@ class GameView(arcade.View):
         """Render the screen."""
         self.clear() # always start with self.clear()

-        arcade.draw_sprite(self.player)
+        self.player_list.draw()
```

Maintenant, créons notre monde.
L'herbe et les buissons seront des éléments qui ne bougeront jamais dans le monde.
Les buissons seront des obstacles, qu'Arcade appele des *walls*.
L'herbe, par contre, est uniquement décorative.

Il est temps de vous faire réellement travailler un peu, donc cette fois nous ne faisons plus que décrire les étapes à réaliser :

1. Créez deux nouvelles `SpriteList` appelées `grounds` et `walls`.
   Construisez celles-ci avec `arcade.SpriteList(use_spatial_hash=True)`.
   Le "spatial hashing" augmente les performances pour une `SpriteList` qui ne contient que des sprites qui ne bougent (presque) jamais.
2. Dans `__init__()`, écrivez deux boucles pour placer les sprites suivants dans cette `wall_list` :
    1. Des sprites d'herbe à toutes les coordonnées de grilles $(x, y)$ pour $x \in \[0, 40\[$ et $y \in \[0, 20\[$.
       Ajoutez-les à la sprite list `grounds`.
    2. Des sprites de buissons aux positions $(x, y)$ dans $\\{(3, 6), (7, 2), (2, 10), (3, 8)\\}$.
       Ajoutez-les à la sprite list `walls`.
3. Dans `on_draw()`, n'oubliez pas de dessiner les deux sprite lists.

Vous devriez obtenir le résultat suivant :

![Le joueur, de l'herbe et des buissons](/assets/img/projet/player-grass-bushes.png)

## Contrôle du joueur au clavier

(Changer de membre du binôme qui "a le contrôle" ici.)

La prochaine étape est de pouvoir déplacer notre joueur.
Pour cela, il va falloir réagir aux appuis sur les touches du clavier.

De la même façon que Arcade appelle `on_draw` pour dessiner l'état du jeu, elle appelle aussi deux méthodes supplémentaires pour le clavier :

* `on_key_press` est appelée lorsqu'une touche est enfoncée ;
* `on_key_release` est appelée lorsqu'une touche est relâchée.

### Mouvement de base

Par exemple, à l'appui sur la touche flèche droite (→), on voudra démarrer un mouvement vers la droite.
Et quand cette touche est relâchée, on veut arrêter le mouvement.

Cela pose la question du *mouvement* !
Comme vous le savez, le mouvement est une évolution de la position dans le temps, qui dépend de la vitesse.
À son tour, la vitesse évolue en dépendant de l'accélération.

Pour un mouvement rectiligne à vitesse constante, nous n'allons pas modéliser l'accélération.
Nous allons directement changer la vitesse en fonction des appuis sur les touches gauche et droite.
Il se trouve que les `Sprite`s d'Arcade possèdent déjà des attributs `change_x` et `change_y` qui représentent la vitesse, comme `center_x` et `center_y` représentent la position.
Nous allons donc en profiter.

Déclarons une constante pour la vitesse latérale du joueur :

```python
PLAYER_MOVEMENT_SPEED = 4
"""Speed of the player, in pixels per frame."""
```

puis ajoutons les méthodes suivantes dans `GameView` :

```python
    def on_key_press(self, symbol: int, modifiers: int) -> None:
        """Called when the user presses a key on the keyboard."""
        match symbol:
            case arcade.key.RIGHT:
                # start moving to the right
                self.player.change_x = +PLAYER_MOVEMENT_SPEED
            case arcade.key.LEFT:
                # start moving to the left
                self.player.change_x = -PLAYER_MOVEMENT_SPEED
            case arcade.key.UP:
                # start moving upwards
                self.player.change_y = +PLAYER_MOVEMENT_SPEED
            case arcade.key.DOWN:
                # start moving downwards
                self.player.change_y = -PLAYER_MOVEMENT_SPEED

    def on_key_release(self, symbol: int, modifiers: int) -> None:
        """Called when the user releases a key on the keyboard."""
        match symbol:
            case arcade.key.RIGHT | arcade.key.LEFT:
                # stop horizontal movement
                self.player.change_x = 0
            case arcade.key.UP | arcade.key.DOWN:
                # stop vertical movement
                self.player.change_x = 0
```

Notez que nous avons défini `PLAYER_MOVEMENT_SPEED` en "pixels per frame".
Une *frame* est la période entre deux calculs de l'évolution du monde + affichage du nouveau monde.
C'est donc bien une *unité de temps*.
Le *pixel* étant notre unité de distance, il est bien vrai que "pixels per frame" est une unité de vitesse.

Arcade suppose plus ou moins qu'il y aura toujours 60 frames par seconde (FPS), donc $1 \text{ frame} = 1/60 \text{ seconde}$, et

$$ 5 \text{ pixels par frame} = 5 \text{ pixels par ($1/60$ seconde)} = 5 \cdot 60 \text{ pixels par seconde} = 300 \text{ pixels par seconde} $$

Mais alors quand modifions-nous la *position* ?
Idéalement, nous voulons le faire une fois par frame.
C'est exactement ce à quoi sert la méthode `on_update`.
Arcade appelle `on_update` une fois par frame, avant d'appeler `on_draw`.

```python
    def on_update(self, delta_time: float) -> None:
        """Called once per frame, before drawing.

        This is where in-world time "advances", or "ticks".
        """
        self.player.center_x += self.player.change_x
        self.player.center_y += self.player.change_y
```

Vous pouvez maintenant lancer le jeu et déplacer le joueur avec les flèches du clavier.

> D'un point de vue mathématico-physique, nous venons d'implémenter (une version rudimentaire) d'un intégrateur numérique.
> Les ajouts de `change_x` à `center_x` par pas de temps (1/60 seconde) reviennent à intégrer numériquement `change_x` par rapport au temps qui passe.
> Les moteurs physiques plus évolués prennent en compte les forces pour calculer les accélérations, puis intègrent celles-ci pour obtenir les vitesses, et intègrent également celles-ci pour obtenir les positions.
> Les nouvelles positions et vitesses sont alors utilisées pour calculer les nouvelles forces, et on répète ce manège.

### Obstacles

Vous aurez remarqué que notre joueur n'est pas du tout arrêtée par les buissons, qui sont pourtant sensés être des obstacles.
En effet, nous n'avons absolument pas tenu compte des collisions dans notre mouvement !

Gérer les collisions est une autre paire de manches.
Heureusement, Arcade nous donne une fonctionnalité toute faite pour gérer ces aspects physiques.
Si ce n'était pas le cas, nous aurions arrêté le projet ici pour cette semaine, et nous aurions passé une semaine rien qu'à gérer les collisions.
Utilisons donc un `arcade.PhysicsEngineSimple` pour gérer la physique de notre joueur.

Il faut le créer dans notre méthode `__init__()`, avec quelques paramètres : le sprite du joueur et une sprite list à considérer comme les *murs* (tout ce qui est solide et empêche le joueur de passer).
On peut ensuite utiliser sa propre méthode `update` à la place de notre calcul.

```diff
diff --git a/gameview.py b/gameview.py
index d42aafa..4a5d7d0 100644
--- a/gameview.py
+++ b/gameview.py
@@ -19,6 +19,8 @@ class GameView(arcade.View):
     grounds: Final[arcade.SpriteList[arcade.Sprite]]
     walls: Final[arcade.SpriteList[arcade.Sprite]]

+    physics_engine: Final[arcade.PhysicsEngineSimple]
+
     def __init__(self) -> None:
         # Magical incantion: initialize the Arcade view
         super().__init__()
@@ -53,6 +55,8 @@ class GameView(arcade.View):
                 scale=SCALE, center_x=grid_to_pixels(x), center_y=grid_to_pixels(y)
             ))

+        self.physics_engine = arcade.PhysicsEngineSimple(self.player, self.walls)
+
     def on_show_view(self) -> None:
         """Called automatically by 'window.show_view(game_view)' in main.py."""
         # When we show the view, adjust the window's size to our world size.
@@ -100,5 +104,4 @@ class GameView(arcade.View):

         This is where in-world time "advances", or "ticks".
         """
-        self.player.center_x += self.player.change_x
-        self.player.center_y += self.player.change_y
+        self.physics_engine.update()
```

Le joueur est désormais correctement bloquée par les buissons.
Ça y est !
Nous avons les bases de nos déplacements.

### Remise à zéro

Il est parfois utile, surtout pendant vos tests, de redémarrer le jeu depuis le départ, dans l'état initial.
C'est justement ce que fait la construction d'une nouvelle `GameView` !
Ajoutez un moyen de redémarrer depuis zéro lors de l'appui sur la touche Escape du clavier.

Consultez [la documentation](https://api.arcade.academy/en/latest/api_docs/arcade.key.html) ou profitez de l'autocomplétion pour découvrir la bonne constante pour cette touche.
Lorsqu'on appuie sur cette touche, créez une nouvelle instance de `GameView`, et appelez la méthode `self.window.show_view` pour remplacer la vue active par la nouvelle.

## Caméra

(Changer de membre du binôme qui "a le contrôle" ici.)

Notre joueur peut maintenant se déplacer.
Mais s'il s'aventure hors des bornes de l'écran, nous ne pourrons pas le suivre.
Normalement, dans un jeu d'aventure, l'écran suit les déplacements du joueur.

Puisqu'on ne veut pas déplacer le monde entier à chaque frame, il est préférable d'introduire une *caméra*.
C'est la caméra qui va bouger, et qui va déterminer quelle portion du monde est affichée à l'écran.
Pour l'instant, nous allons nous contenter d'une caméra très simple qui est constamment centrée sur le joueur.

Créons d'abord une `arcade.camera.Camera2D`, et utilisons-là pour dessiner tous les sprites de notre monde.

```diff
diff --git a/gameview.py b/gameview.py
index adb61bc..be39244 100644
--- a/gameview.py
+++ b/gameview.py
@@ -20,6 +20,7 @@ class GameView(arcade.View):
     walls: Final[arcade.SpriteList[arcade.Sprite]]

     physics_engine: Final[arcade.PhysicsEngineSimple]
+    camera: Final[arcade.camera.Camera2D]

     def __init__(self) -> None:
         # Magical incantion: initialize the Arcade view
@@ -56,6 +57,7 @@ class GameView(arcade.View):
             ))

         self.physics_engine = arcade.PhysicsEngineSimple(self.player, self.walls)
+        self.camera = arcade.camera.Camera2D()

     def on_show_view(self) -> None:
         """Called automatically by 'window.show_view(game_view)' in main.py."""
@@ -69,9 +71,10 @@ class GameView(arcade.View):
         """Render the screen."""
         self.clear() # always start with self.clear()

-        self.grounds.draw()
-        self.walls.draw()
-        self.player_list.draw()
+        with self.camera.activate():
+            self.grounds.draw()
+            self.walls.draw()
+            self.player_list.draw()

     def on_key_press(self, symbol: int, modifiers: int) -> None:
         """Called when the user presses a key on the keyboard."""
```

Pour l'instant, rien ne se passe, car nous ne faisons pas bouger la caméra.
On veut faire cela dans la méthode `on_update`, après avoir appliqué la physique du jeu.

```diff
diff --git a/gameview.py b/gameview.py
index 5c715eb..be39244 100644
--- a/gameview.py
+++ b/gameview.py
@@ -111,3 +111,4 @@ class GameView(arcade.View):
         This is where in-world time "advances", or "ticks".
         """
         self.physics_engine.update()
+        self.camera.position = self.player.position
```

La caméra suit désormais le joueur.

Cela révèle malheureusement qu'on peut se déplacer en dehors des limites de notre monde.
Pouvez-vous ajouter des buissons tout le long des bords du monde pour éviter cela ?

## Animations

On l'a dit plus tôt, les assets que l'on vous fournit sont, pour la plupart, des *animations*.
C'est notamment vrai pour l'image du joueur.
Là aussi, Arcade nous aide, avec les concepts de `TextureAnimation` et `TextureAnimationSprite`.

Voici un dernier bout de boilerplate que vous pouvez copier-coller et ajouter à `textures.py` :

```python
def _load_animation_strip(
    file: str,
    frame_count: int,
    frame_duration: int = 100,
    tile_size: tuple[int, int] = ORIG_TILE_SIZE,
) -> arcade.TextureAnimation:
    """
    Loads an animation strip from a line-oriented spritesheet.

    Args:
        file:
            Path to the spritesheet file name.
        frame_count:
            The number of frames in the animation, which should also be the
            number of sub-images in the file.
        frame_duration (optional):
            The duration of each frame in ms (defaults to 100).
        tile_size (optional):
            The size in pixels of one element of the grid, i.e.,  of a frame.
            Defaults to the standard tile size of `(16, 16)` that we use in our
            assets.

    Returns:
        An `arcade.TextureAnimation` representing the full animation.
    """
    grid = _load_grid(file, columns=frame_count, rows=1, tile_size=tile_size)
    keyframes = [arcade.TextureKeyframe(frame, frame_duration) for frame in grid]
    return arcade.TextureAnimation(keyframes)
```

Remplacez la définition de `TEXTURE_PLAYER_IDLE_DOWN` par :

```python
ANIMATION_PLAYER_IDLE_DOWN: Final[arcade.TextureAnimation] = \
    _load_animation_strip("assets/Top_Down_Adventure_Pack_v.1.0/Char_Sprites/char_idle_down_anim_strip_6.png", 6)
```

`ty` devrait vous signaler une erreur à la création de `self.player` dans `gameview.py`.
Au lieu de définir un simple `arcade.Sprite`, nous allons utiliser un `arcade.TextureAnimationSprite` :

```diff
diff --git a/gameview.py b/gameview.py
index be39244..b744b9e 100644
--- a/gameview.py
+++ b/gameview.py
@@ -13,8 +13,8 @@ class GameView(arcade.View):
     world_width: Final[int]
     world_height: Final[int]

-    player: Final[arcade.Sprite]
-    player_list: Final[arcade.SpriteList[arcade.Sprite]]
+    player: Final[arcade.TextureAnimationSprite]
+    player_list: Final[arcade.SpriteList[arcade.TextureAnimationSprite]]

     grounds: Final[arcade.SpriteList[arcade.Sprite]]
     walls: Final[arcade.SpriteList[arcade.Sprite]]
@@ -33,8 +33,8 @@ class GameView(arcade.View):
         self.world_width = 40 * TILE_SIZE
         self.world_height = 20 * TILE_SIZE

-        self.player = arcade.Sprite(
-            TEXTURE_PLAYER_IDLE_DOWN,
+        self.player = arcade.TextureAnimationSprite(
+            animation=ANIMATION_PLAYER_IDLE_DOWN,
             scale=SCALE, center_x=grid_to_pixels(2), center_y=grid_to_pixels(2)
         )
         self.player_list = arcade.SpriteList()
```

Si vous lancez le jeu à ce stade, rien n'a changé.
Il faut encore demander à l'animation de se mettre à jour lors de `on_update` :

```diff
diff --git a/gameview.py b/gameview.py
index b744b9e..d4f5323 100644
--- a/gameview.py
+++ b/gameview.py
@@ -111,4 +111,5 @@ class GameView(arcade.View):
         This is where in-world time "advances", or "ticks".
         """
         self.physics_engine.update()
+        self.player.update_animation()
         self.camera.position = self.player.position
```

Et voilà, vous avez maintenant un joueur animé !

## Ramasser des cristaux

(Changer de membre du binôme qui "a le contrôle" ici.)

Nous arrivons au dernier concept important à connaître avant de pouvoir vous lâcher dans la nature d'Arcade : la détection de collisions.
Certes, le moteur physique s'occupe des collisions entre le joueur et les *murs*, dans le but de contrôler ses mouvements.
Mais nous devrons aussi gérer les collisions pour d'autres aspects du jeu, notamment pour ramasser des objets.
En effet, on ne veut pas empêcher le joueur de se rendre sur un objet.
Au contraire, on souhaite qu'il le fasse, pour le ramasser et lui remettre.

Pour cela, nous allons utiliser des crystaux à collectionner.
Commencez par ajouter quelques crystaux dans le monde :

1. Définissez une nouvelle `TextureAnimation` pour les crystaux, chargée depuis l'asset `"assets/Top_Down_Adventure_Pack_v.1.0/Props_Items_(animated)/crystal_item_anim_strip_6.png"`.
2. Créez une nouvelle sprite list appelée `crystals` (les crystaux étant fixes, activez le "spatial hashing" comme pour les murs).
3. Remplissez-là avec des crystaux, aux positions $(5, 2), (6, 5), (3, 5)$.
4. N'oubliez pas de les dessiner dans `on_draw`, et de mettre à jour leur animation dans `on_update`.

Puisque c'est une sprite list différente des murs, le moteur physique permet au joueur de passer dessus.
Mais il faut encore les ramasser.

Pour cela, dans notre méthode `on_update`, nous allons tester si le joueur est "sur" un crystal.
Si c'est le cas, nous allons le retirer du monde, ce qui donnera l'illusion que le joueur l'a ramassé.
Plus tard, vous pourriez compter les crystaux ramassés dans un score, par exemple.

Pour tester si le joueur est sur un crystal, nous devons en fait tester si le sprite du joueur est en *collision* avec le sprite d'un des crystaux.
C'est une opération fréquente, et coûteuse si on ne s'y prend pas bien.
Heureusement, Arcade nous fournit exactement ce dont nous avons besoin : la fonction

```python
arcade.check_for_collision_with_list(self.player, self.crystals)
```

renvoie la liste de toutes les sprites dans `self.crystals` qui sont en collision avec `self.player`.
En pratique, cette liste contiendra 0 ou 1 élément, mais pourquoi ne pas la gérer de manière générique ?

L'autre méthode dont avez besoin est

```python
some_sprite.remove_from_sprite_lists()
```

qui retire `some_sprite` de toutes les sprite lists dans lesquelles elle se trouve.
En pratique, dans notre cas, cela sert à retirer un crystal de la liste `self.crystals`.

Utilisez ces deux méthodes dans `on_update` pour permettre au joueur de ramasser les crystaux qu'il touche.

### Astuce : afficher les "hit boxes"

Il y a des algorithmes avancés derrière la fonction `check_for_collision_with_list`, qui sortent du cadre de ce cours.
Cependant, il est bon de savoir qu'ils se basent sur la notion de "hit box".
Idéalement, on voudrait que deux sprites entrent en collision si et seulement si il existe un pixel en commun dans leurs images.
En pratique, cela est trop coûteux.

Arcade calcule donc une "hit box" pour chaque sprite : un polygone, souvent un octogone irrégulier, qui circonscrit l'image tout en l'approchant le plus possible.
L'algorithme de collisions n'a plus alors qu'à tester si les octogones se superposent, ce qui est beaucoup plus efficace que des formes arbitraires.

Vous pouvez demander à Arcade de dessiner les hit boxes des sprites d'une sprite list avec

```python
some_sprite_list.draw_hit_boxes()
```

dans votre méthode `on_draw`.
Essayez !

## Tests

(Changer de membre du binôme qui "a le contrôle" ici.)

Les jeux vidéos sont, de notoriété publique, difficiles à tester.
En effet, la "sortie" du programme n'est autre qu'un flux vidéo, dont il faudrait en théorie tester précisément certains pixels.
C'est évidemment très fastidieux, et très fragile : de petits changements de traitements sans conséquence peuvent casser des tests pour rien.

On va donc plutôt choisir de tester certains *états internes* de notre jeu.
Par exemple, au lieu de tester que le pixel $(300, 200)$ est de la bonne couleur, on va tester que la position du joueur est bien (proche de) $(300, 200)$.
Pour cela, il faut avoir directement accès à cet état interne depuis nos tests.
Si vous vous demandiez pourquoi nous avons défini tous nos attributs de manière publique, c'est pour ça.

Voyons donc comment tester que notre joueur est capable de ramasser des crystaux.

### Préliminaires : `conftest.py`

Arcade a une limitation majeure : il ne peut y avoir qu'une seule `arcade.Window` dans le programme.
Il n'est même pas possible d'en "détruire" une pour pouvoir en créer une autre.
Il va donc falloir tricher dans nos tests pour que tous les tests réutilisent la même `Window`.
Heureusement, chaque test pourra créer sa propre `GameView`, donc ce ne sera pas trop gênant.
La partie délicate est de faire en sorte que les tests aient *accès* à une `Window` globale, qui sera "remise à zéro" avant chaque test.

Cela étant vraiment compliqué à faire, on vous le donne.
Téléchargez le fichier [`conftest.py`](./conftest.py) et sauvegardez-le dans votre projet sous `tests/conftest.py`.
Grâce à l'existence de ce fichier, vos tests pourront demander accès à la `Window` globale avec un paramètre, sous la forme :

```python
def test_something(window: arcade.Window) -> None:
    # use the window in the test
```

On ne vous demande pas de comprendre le contenu de ce fichier.
Je ne l'ai pas écrit moi-même.
J'ai récupéré la mécanique utilisée par la bibliothèque Arcade elle-même, et l'ai simplifiée un tout petit peu.

### Un test

Voici un test pour s'assurer que le joueur peut ramasser des crystaux :

```python
import arcade

from gameview import GameView

def test_collect_crystals(window: arcade.Window) -> None:
    view = GameView()
    window.show_view(view)

    INITIAL_CRYSTAL_COUNT = 3

    # Make sure we have the amount of coins we expect at the start
    assert len(view.crystals) == INITIAL_CRYSTAL_COUNT

    # Start moving right
    view.on_key_press(arcade.key.RIGHT, 0)

    # Let the game run for 1 second
    window.test(60)

    # We should have collected the first coin
    assert len(view.crystals) == INITIAL_CRYSTAL_COUNT - 1

    # Stop moving right, move up
    view.on_key_release(arcade.key.RIGHT, 0)
    view.on_key_press(arcade.key.UP, 0)

    # Let the game run for 1 more second
    window.test(60)

    # We should have collected the second coin
    assert len(view.crystals) == INITIAL_CRYSTAL_COUNT - 2
```

La méthode `window.test(frames)` nous est offerte par Arcade pour faciliter l'écriture de tests.
Elle génère les appels à `on_update` et `on_draw` pendant le nombre de frames spécifié en argument.
D'une certaine manière, on peut considérer qu'elle *fait avancer le temps*.

Les tests qui auront besoin d'une `Window` vont suivre la formule suivante :

1. Créer une `view: GameView` (potentiellement avec des paramètres, dans le future).
2. Utiliser `window.show_view(view)` pour installer cette vue dans l'unique fenêtre de test.
3. Simuler un scénario avec un mélange de :
    * `window.test(frames)` pour faire avancer le temps.
    * Appels à `view.on_key_press`, `on_key_release` pour simuler les appuis de touches.
    * Modifier directement `view.player.position` pour "téléporter" le joueur (au lieu de naviguer par touches).
    * Assertions qui testent l'état interne attendu à certains moments dans le temps.

Vous pourrez prendre exemple sur le test ci-dessus pour toutes vos fonctionnalités supplémentaires, en commençant par les améliorations de la prochaine section.

## Améliorations

**À partir d'ici, vous pouvez travailler séparément.**

Nous vous proposons quelques améliorations à l'ergonomie de votre jeu, qui se basent sur les concepts que vous avez déjà en main.

Répartissez-vous le travail !

### Meilleure gestion du clavier

Avez-vous remarqué que votre joueur s'arrête parfois alors que vous appuyez encore sur une touche ?
Cela peut se produire, par exemple, avec la séquence suivante :

1. Enfoncer la touche droite
2. Enfoncer la touche gauche
3. Relâcher la touche droite

Le joueur s'arrête, alors qu'il devrait continuer vers la gauche.

**Identifiez** le problème et **corrigez**-le.

### Gestion plus fine de la caméra

Souvent dans un jeu de plateformes, la caméra ne reste pas exactement centrée sur le joueur.
Le joueur peut se déplacer dans les quatre directions sans que la caméra ne bouge.
La caméra ne se met à bouger que lorsque le joueur s'approche "trop près" des bords.
De plus, peu importe à quel point le joueur s'approche du bord, la caméra limite son déplacement pour ne jamais montrer l'extérieur du monde (vous avez déjà dû remarquer que pour l'instant, elle nous montre le "bleu" de l'extérieur du monde, ce qui n'est pas très élégant).

**Adaptez** la gestion de la caméra pour suivre ce nouveau modèle.
Essayez différentes valeurs de "marges", c'est-à-dire combien de pixels doivent être visibles autour du joueur au minimum.
Dans tous les cas, les bords de la caméra ne doivent pas dépasser les bords du monde (rappelez-vous les attributs `world_width` et `world_height`).

### Bruitages

Un jeu sans bruitages, c'est un peu monotome.
**Ajoutez** des sons lors de la collecte des crystaux.

Vous aurez besoin des fonctions suivantes :

* [`arcade.load_sound`](https://api.arcade.academy/en/latest/api_docs/api/sound.html#arcade.load_sound) (en mode `streaming=False`, par défaut)
* [`arcade.play_sound`](https://api.arcade.academy/en/latest/api_docs/api/sound.html#arcade.play_sound) (en mode `loop=False`, par défaut)

Il y a [des sons disponibles ici](https://api.arcade.academy/en/latest/api_docs/resources.html#sounds), qui sont automatiquement mis à disposition de votre jeu par Arcade.
Nous suggérons :

* `":resources:sounds/coin5.wav"` pour ramasser un crystal

Faites en sorte de ne charger (`load_sound`) chaque son qu'une seule fois, et non pas à chaque fois que vous voulez le jouer.
Inspirez-vous de la façon dons vous avez chargé les textures et les animations.

Vous pourriez aussi naviguer dans la documentation de Arcade pour voir s'il y a plus d'explications ou d'exemples sur l'utilisation des sons.

**Remarque :** il n'est pas possible de tester automatiquement les sons ; nous ne nous attendons donc pas à ce que vous le fassiez.

## Compléter le log

N'oubliez pas de compléter [votre fichier `LOG.md`](./#rendu).

Il n'y a pas de "question de design" cette semaine.
