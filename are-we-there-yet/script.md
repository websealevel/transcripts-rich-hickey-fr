# Traduction et transcription de la conférence *Est-ce déjà la fin ? Une déconstruction de la perception du temps en tant qu'objet* de Rich Hickey

Titre original : Are We There Yet ?

[Lien vers la conférence](https://www.infoq.com/presentations/Are-We-There-Yet-Rich-Hickey/)

<!-- traductions possibles : Avons nous atteint notre but ? Est-ce déjà la fin de l'histoire ? Est-ce déjà la fin ? Sommes nous arrivés au bout ? -->

Version: 1.0

Date dernière mise à jour:

## *Est-ce déjà la fin ? Une déconstruction de la perception du temps en tant qu'objet* de Rich Hickey

Je vais parler du *temps* aujourd'hui. Plus particluièrement, de comment on gère le temps dans les langages orienté objet en général et peut-être de comment nous échouons à le faire.

<!-- <img src="assets/slide00.png" width="300px"> -->

Mon but aujourd'hui est de vous provoquer, de vous pousser à reconsidérer des choses fondamentales avec lesquelles, je pense, nous sommes tellement en contact chaque jour que nous ne les voyons plus si bien que nous n'arrivons pas à prendre le recul nécessaire sur ce que nous faisons *vraiment*.

Donc, sommes nous bien équipés avec les langages OOP tels que nous les connaissons aujourd'hui ? Le concept est assez large, et il y a un certain nombre d'implémentations mais tous partagent un certain nombre d'attributs. Est ce que nous sommes tous d'accord qu'il s'agit là de la meilleure manière d'écrire des logiciels ? Est ce que ce sera la meilleure manière dans le futur ?

<!-- <img src="assets/slide01.png" width="300px"> -->

A l'heure actuelle, il s'agit d'un modèle complètement retranché sur lui même. Peu importe le langage que vous utilisez, chaque langage a ses petites particularités comme Scala, Java, ou C# et les gens adorent les différences entre ces langages mais je voudrais que vous vous concentriez plutot sur les similarités entre ces langages. Tous ces langages sont single-dispatch, fondé sur des états gérés par des objets. Ils disposent tous des mêmes genres de choses: des classes, de l'héritage, des attributs (concept intéressant), des méthodes (encore plus intéressant, on y reviendra), ils disposent tous d'un ramasse-miettes. Ils ont tous pour ancêtre commun le langage SmallTalk.

<!-- <img src="assets/slide02.png" width="300px"> -->

Tous ces langages ne sont pas significativement différents. Ils sont différents de manière superficielle. Ils peuvent avoir des mixins, des interfaces, être typés statiquement ou dynamiquement, etc. Mais aucun de ces aspects n'est vraiment important comparé à l'ADN commun qu'ils partagent. Tout le monde est excité car leur langage n'a pas de point virgules etc. Mais toutes ces différences ont plus à voir avec la sensibilité du programmeur, elles ne sont significatives quant au *modèle* de programmation qui reste toujours le même. Ce sont toutes des voitures différentes, mais elles roulent toutes sur la même route.


<!-- <img src="assets/slide03.png" width="300px"> -->

Est-ce la fin ? Sommes nous arrivés au bout ? Allons nous continuer à inventer des langages qui n'évoluent que par des petites variations incrémentales infinitésimales par rapport aux choses que nous connaissons déjà ? Ce qui est indéniable, c'est que les gens aiment l'orienté objet. D'un autre côté, je pense qu'on est devenus profondémment conservateurs. Et c'est compréhensible, vous avez été recruté par des grandes entreprises, elles ont investi beaucoup dedans, les gens savent s'en servir etc. Ce n'est pas quelque chose dont on se sépare si facilement. 

Et je voudrais insister sur le fait que le but de cette présentation n'est pas de taper sur l'orienté objet, mais plutot de vous inviter à faire un pas de côté, que vous imaginiez que vous n'aimez pas l'orienté objet et que vous vous demandiez si c'est vraiment un modèle parfait.

<!-- <img src="assets/slide04.png" width="300px"> -->

Quand nous regardons les langages et que nous nous disons "ah, si je pouvais écrire un autre langage" ou "si je pouvais faire quelque chose pour corriger ce langage", ou encore "si je pouvais ajouter une nouvelle fonctionnalité à mon langage", etc. que faisons-nous ? Pourquoi ajoutons nous quelque chose ? Qu'est ce qui nous pousse à faire des changements ? Qu'est ce qui nous pousse à changer de voiture par exemple ? Qu'est ce qui nous pousse à adopter un autre langage ? Et qu'est ce qui *devrait* nous pousser à changer ? Je ne pense pas qu'il y ait beaucoup de personnes qui disent "Oh, je suis fatigué des point-virgule, je ne peux plus en écrire. Je vais passer à quelque chose de plus simple". Je pense que le typage dynamique peut pousser des gens à changer mais je pense qu'il y a déjà des exemples dans l'Histoire qui illustrent ce qui nous pousse à changer. 

<!-- <img src="assets/slide05.png" width="300px"> -->

Je vais donc parler aujourd'hui d'un petit sous-ensemble de choses auxquelles vous devriez penser quand vous regardez le langage que vous utilisez, et que vous vous demandez si vous ne pouvez pas faire autrement.

Je veux parler de complexité et de temps, surtout de la notion de temps. Enfin, je veux parler de modèles que nous pouvons utiliser pour mieux implémenter le temps et de certains des principes sur lesquels reposent l'orienté objet. L'orienté objet est une manière de modéliser, basé sur l'idée que nous pouvons faire des choses similaires dans nos programmes à ce que nous faisons, voyons dans la vraie vie, et que ça nous aide à comprendre nos programmes.

<!-- <img src="assets/slide06.png" width="300px"> -->

Le héros de cette présentation est [Alfred North Whitehead](https://fr.wikipedia.org/wiki/Alfred_North_Whitehead), c'est le type célèbre qui avec [Russel](https://fr.wikipedia.org/wiki/Bertrand_Russell) à écrit [Principa Mathematica](https://fr.wikipedia.org/wiki/Principia_Mathematica). Par la suite, il est dévenu philosophe et il a écrit des super trucs et je vais en citer quelques passages, car ils sont tops. 

>Seek simplicity, and distrust it (Recherche la simplicité, et méfie en toi)

La première chose est de se méfier de la simplicité. Je ne parle pas ici de la complexité des problèmes que nous essayons de résoudre. Nous savons tous qu'on nous donne des problèmes de plus en complexes à résoudre, des problèmes toujours plus gros, avec toujours plus de données, pour lesquels la solution doit être de plus en plus flexible. Les gens vont toujours attendre davantage des logiciels. 

<!-- <img src="assets/slide07.png" width="300px"> -->


La complexité dont je veux parler aujourd'hui est la *complexité circonstancielle* _(NDLR *incidental complexity* en anglais)_, la complexité qui naît de la manière dont nos outils, et les idées incarnées par ces outils, fonctionnent ou ne fonctionnent pas. Ces choses deviennent des problèmes que nous devons résoudre et vous avez un nombre limité d'heures par jour, et vous devez résoudre des problèmes. Est ce que ces problèmes à résoudre sont des problèmes du domaine d'application ou ceux que vous avez érigé vous même en choisissant tel outil, tel langage particulier ou telle stratégie de développement ? C'est ça la complexité circonstancielle, elle monte toujours à bord, elle ne fait pas partie du problème que vous essayez de résoudre. 

<!-- [5:35](https://youtu.be/E4RarTAZ2AY?t=335) -->

>La complexité circonstancielle la plus redoutable, c'est celle qui revêt l'apparence de la *simplicité*


Et c'est la pire je pense, on ne sait jamais quand on fait quelque chose de complexe, il n'y a pas un truc qui apparait soudainement et s'exclame "Arrg, complexe !", qui vous pousse à vous dire "Oh, d'accord je vois la complexité, elle est effrayante, je sais que je viens d'entrer dans une zone dangereuse, je dois rester prudent là dessus." La complexité circonstancielle la plus redoutable, c'est celle qui revêt l'apparence de la *simplicité*. "Regarde comment c'est facile, il n'y a pas de points-virgules etc.". Je ne dis pas ça pour critiquer les points-virgules, c'est juste quelque chose qui m'incite à regarder des aspects superficiels du langage que j'utilise, qui me pousse à me dire "ça à l'air simple, ça m'est familier" (*NDLR ici Rich Hickey ne semble pas vouloir aborder la distinction entre la simplicité et la facilité, un thème qu'il va approfondir dans sa conférence [Simple made Easy](https://www.youtube.com/watch?v=kGlVcSMgtV4)*). La question est de savoir est-ce que *derrière ces apparences* se cache une complexité circonstancielle ?

Prenons un exemple _(NDLR un exemple tiré du C++)_

~~~C++
Foo *bar(...){...}; //quel est le problème ?
~~~

et je le répète, je ne cherche pas à critiquer gratuitement le C++ mais j'ai passé plus d'une décennie à en faire... Bon, ce n'est pas si difficile, si vous êtes versé dans la [métaprogrammation avec des patrons](https://fr.wikipedia.org/wiki/M%C3%A9taprogrammation_avec_des_patrons) ça peut devenir difficile, mais les bases sont plutôt simples: vous pouvez écrire une fonction, elle retourne un pointeur. Qu'est ce qui ne va pas avec ça ? Parce que c'est plutôt simple: il y a `new` et `delete` _(NDLR new et delete sont des opérateurs de la librairie standard qui permettent d'allouer et de désallouer de la mémoire en C++)_, des pointeurs que vous pouvez faire circuler dans votre programme, les déférencer _(NDLR accèder à la valeur stockée à l'adresse mémoire stockée par le pointeur)_, etc. Vous avez besoin de connaître cinq choses au plus et vous pouvez les apprendre en une après-midi. 

Mais pour autant, est-ce *vraiment simple* ? Par exemple, la même syntaxe est utilisée pour les pointeurs qui font référence à des choses sur le [tas](https://fr.wikipedia.org/wiki/Tas_(allocation_dynamique)) et des choses qui ne sont pas sur le tas. Mais il y a pire, le vrai problème qui réside dans la signature de cette fonction c'est qu'est ce que vous faites du résultat qu'elle vous renvoie ? Est-ce que ça vous appartient ? En êtes vous responsable à présent ? Devez vous libérer la mémoire plus tard vous même _(NDLR utiliser l'opérateur `delete`)_ ? Est-ce quelque chose dont la mémoire peut être libérée ? Pouvez-vous le passer à quelqu'un d'autre ? Es-ce autorisé ? Pouvez vous l'enregistrer ? Le problème ici c'est qu'il n'y a pas de de gestion standardisée et automatique de la mémoire. Il n'y a pas de [ramasse-miettes](https://fr.wikipedia.org/wiki/Ramasse-miettes_(informatique)). Et pour celleux qui utilisent ce langage, ce problème était (et l'est toujours) une grande source de complexité circonstancielle. Car *vous* devez gérer la mémoire. Et vous ne le voyez pas immédiatement, il n'y pas un panneau dans l'en tête de votre code source qui vous avertit "N'oubliez pas qu'il est de votre responsabilité de gérer la mémoire!". C'est de la complexité circonstancielle, vous *devez le savoir*, ce n'est pas écrit dans le code source. 

Je pense d'ailleurs que l'absence de ramasse-miettes a sérieusement entravé un des objectifs principaux du C++, à savoir celui d'être un langage de [bibliothèque logicielle](https://fr.wikipedia.org/wiki/Biblioth%C3%A8que_logicielle) *(NDLR library language dans le texte)*. Toutes les intentions initiales du langage, toutes les interventions de [Stroustrup](https://fr.wikipedia.org/wiki/Bjarne_Stroustrup) (NDLR Bjarne Stroustrup est l'auteur du C++) etc., soulignent cette volonté du C++ de devenir un langage de bibliothèqe logicielle mais il a fini seulement par devenir un langage de bibliothèque étriqué. Car chacun dispose de sa propre bibliothèque, mais il n'existe toujours pas beaucoup de bibliothèques partagées entre différents acteurs à cause de ce problème. Et nous savons que Java, qui dispose d'un ramasse-miettes standard, a réussi à créer une grande infrastructure de bibliothèque logicielle. Je pense que ce qui a incité beaucoup de gens à migrer de C++ vers Java, en grande partie, est de ne plus avoir à porter le fardeau de cette complexité. "Je ne veux plus faire de gestion de mémoire manuelle, cela ne fait pas partie du problème que j'essaie de résoudre. C'est juste un problème de plus, qui m'est resservi jour après jour, dès que je me remets au travail. Et je n'en veux plus." 





## Bibliographie

- [Are we there yet ?](https://www.youtube.com/watch?v=E4RarTAZ2AY&t=3299s), enregistrée en 2009, au Sun JVM Languages Submit, filmée en partenariat avec [InfoQ.com](https://www.infoq.com/). La source de cette transcription.
- [Simple made Easy](https://www.youtube.com/watch?v=kGlVcSMgtV4), enregistrée en 2011, mise à disposition par [InfoQ.com](https://www.infoq.com/)


## Auteur

Paul Schuhmacher

site web: <a href="pschuhmacher.com">pschuhmacher.com</a>

contact: <a href="mailto:contact@pschuhmacher.com">contact@pschuhmacher.com</a>