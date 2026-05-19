---
title: "Série 11 : Itérateurs et générateurs"
layout: default
---

# Série 11 : Itérateurs et générateurs

[Solutions](https://github.com/epfl-cs-112-ma/solutions-serie-11)

Cette série a pour objectif de pratiquer l'usage des itérateurs et générateurs.

Avant de commencer cette série, il est prévu que vous ayez suivi le tutoriel de cette semaine :

* [Itérateurs et générateurs](/tutoriels/generateurs.html)

Nous vous recommandons de créer [un nouveau projet Python](/references/quick-projet-setup.html) pour chaque série d'exercices.
Cela vous permettra d'isoler plus facilement le contenu des différentes semaines.

Cette série contient deux exercices :

{::options toc_levels="2" /}

* This will become a table of contents (this text will be scrapped).
{:toc}

## Itérateur avec indices

Écrivez une classe `StrIterator` qui soit un `Iterator[tuple[int, str]]`.
Elle reçoit en argument du constructeur une chaîne `s: str`.
Itérer sur un tel itérateur doit donner successivement des paires `(i, c)` où `i` est un index dans la chaîne, et `c` est le caractère à cet index.

Exemple d'utilisation :

```python
it = StrIterator("foobar")
for i, c in it:
    print(f"{i}: {c}")
```
affichera
```
0: f
1: o
2: o
3: b
4: a
5: r
```

Une `str` est un `Iterable[str]`.
Généralisez votre `StrIterator` en `IndexIterator[T]` pour fonctionner sur n'importe quel `Iterable[T]`, produisant des `tuple[int, T]`.

Écrivez maintenant une fonction `index_iterator[T]` qui accepte un `Iterable[T]` et renvoie un `Iterator[tuple[int, T]]`.
L'itérateur renvoyé doit itérer de la même façon que `IndexIterator`.
Une implémentation valable de cette fonction serait de renvoyer un `IndexIterator`.
Écrivez-en une autre version, qui soit écrite sous forme de *fonction génératrice*, avec `yield`.

## Générateur de positions

Supposons qu'on ait une carte de largeur `width` et hauteur `height`.
On souhaite itérer sur toutes les positions `(x, y)` valides de cette carte.

Écrivez une fonction qui accepte `width` et `height`, et qui renvoie un `Iterator[tuple[int, int]]`.
Faites-en 3 versions :

* une avec une fonction génératrice ;
* une avec une generator expression ;
* une "à la main", avec une classe qui étend `Iterator[tuple[int, int]]`.

Assurez-vous qu'elles renvoient des résultats équivalents.

Laquelle préférez-vous ?
