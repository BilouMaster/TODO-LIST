![exemple](http://biloucorp.com/BCW/Joke/exemple_de_reorganisation.png)

(J'ai oublié de déplacer **EventPrinter** dans **RME-samples**... mon bel exemple est foutu en l'air !)

(╯°□°）╯︵ ┻━┻



RME est déjà divisé en plusieurs scripts, et bien maintenant si on le voulait, on pourrait le découper de manière organisée sous forme de **modules** (au sens général, pas au sens ruby) comme dans cet exemple... Et créer de nouveaux modules plus tard si on est super op, que ce soit des modules *"développeurs/contributeurs"* comme **RME-doc** et **RME-samples**, ou des modules *"utilisateurs"* comme l'est actuellement **RME**.

De la même manière que chaque script se suit actuellement selon s'il est dépendant ou non du précédent, chaque module se suivrait selon s'il est dépendant ou non du précédent.

Par exemple, il me semble que le module **RME-samples** est indépendant, on le charge avant ou après n'importe quel autre module, mais le module **RME-doc** est dépendant du module **RME**, donc il sera chargé après le module RME, tel que défini dans le **_list.rb** de src.

Une fois qu'on sait travailler en modules *(si jamais vous me suivez jusque là...)*, on peut faire des sous-modules (sous-dossiers)... mais c'est une autre histoire... On ne va pas développer UN PROGRAMME non plus. :v

En y pensant, avec les sous-modules on pourrait *s'éclater* à découper d'avantage les scripts actuels. Par exemple avoir **RME/Commands/** au lieu de **Commands.rb**, dossier qui contiendrait chaque catégorie de commande sous forme de script individuel... Et bim, c'est plus facile à maintenir, ça respecte le design global, et on peut de surcroît ré-utiliser cette structure pour le futur **doc-generator**.

Et le **package.rb**... pour l'heure il ne sert qu'à compiler **RME.rb**, si on partait en freestyle organisationnel avec des sous-modules, il faudrait compiler **RME.rb** en faisant une simple récursion sur les **_list.rb**... en fait je pourrais améliorer mon **scripts-compiler** en lui ajoutant un paramètre de configuration `COMPILE_TO = :rvdata` par défaut mais qui pourrait être utilisé comme `COMPILE_TO = 'chemin/vers/un/script.rb'` si on veut... x)

Un autre avantage avec *ce genre de* découpage, c'est qu'on a le choix entre charger le dossier **src** ou charger uniquement le dossier **src/RME**, sachant que **scripts-loader** peut charger le **_list.rb** de n'importe quel dossier (distant ou non)

Par exemple un projet en dehors du repo GitHub pourrait charger RME si on utilise **scripts-loader** avec `LOAD_FROM = 'C:/super/chemin/vers/le/dossier/RME/'`

Alors bien sûr comme on a déjà notre **RME.rb** de compilé et on à qu'à faire un bon vieux `Kernel.send(:load, 'C:/super/chemin/vers/RME.rb')` comme je le fais moi-même, mais bon je me suis permis de diverger.

Je me permet encore de diverger en ajoutant qu'on peut aussi remplacer le `$STAGING = true` par le choix de charger `"../src"` ou `"../src/RME"` dans notre `RME/project`

Pour en revenir au `COMPILE_TO = :rvdata` du **scripts-compiler**, comme ça risque de compiler une grosse floppée de merdier dans **Scripts.rvdata2** si on fait des sous-modules, je pense à étendre l'interprétation du **_list.rb** en permettant d'ajouter un premier élément "`<compact!>`" dans la liste, destiné au compilateur, qui lui donnerait l'instruction de compacter toute la liste des scripts et modules (dossiers) en un seul script qui porterait simplement le nom du dossier. En bref on pourrait compiler notre arborescence dans un seul script "**RME**" directement dans le **Scripts.rvdata2** (ce qui n'exclue pas l'idée de continuer à compiler le **RME.rb** actuel avec `COMPILE_TO = '../RME.rb'`, ce qui est plus user-friendly.

(Je précise que pour comprendre mon délire, il faut déjà comprendre le fonctionnement actuel des trois scripts du repo https://github.com/RMEx/scripts-externalizer... pour ceux qui débarquent)

Toujours dans l’idée de fractionner le tout en modules et sous-modules *a notre guise*, je pourrais ajouter au **_list.rb** un élément optionnel genre « <require nom_de_module> » ou « `<require nom_du_script.rb>` » destiné à **scripts-loader** et **scrips-compiler** pour que ces derniers renvoient une erreur si on charge/compile dans un mauvais ordre ou qu’on charge/compile un (sous-)module tout seul sans faire attention à ses dépendances. ça permettrait de laisser une trace de notre organisation  (« pourquoi on les a mis dans cet ordre là ») quelque-part. On pourrait aussi avoir l’équivalent ruby genre `XT.require()` qui servirait à déclarer non pas pour un module (**_list.rb**) entier mais pour un script individuel... `<require x>` et `XT.require()`, contrairement au Kernel.require, serviront juste à arrêter la compilation/le chargement avec un message d’erreur custom s’ils ne sont pas satisfaits.

Par exemple XT.require("SDK.rb") (script requis) ou XT.require("RME") (module requis) au début d’un script, juste pour vérifier si X est bien chargé/compilé, pas pour charger puisque c’est le taf du scripts-loader

La différence avec le « require » de ruby, c’est que XT.require peut requérir un module (toujours au sens « dossier » ou plus précisément « _list.rb »), faut que les XT.require soient par exemple mis en commentaire par **scripts-compiler** et remis en « non-commentaire » par **scripts-externalizer**

En fait je réinvente le concept de librairie externe... version compilable vers scripts.rvdata2 ou vers un script (RME.rb)

On virerait tous les commentaires, etc pour optimiser le poids et tout

Ça ferait une troisième manière de releases notre programme

Et quatre manière d’installer RME : RME.rb, ou RME.rvdata2, ou RME-external qui lui-même s’utiliserait en « load » ou « compile »

ça se résumerait par exemple à faire `LOAD_FROM = 'Scripts/RME.rvdata2'` avec **scripts-loader**, ou bien `COMPILE_TO = ['../RME.rb', '../RME.rvdata2']` avec **scripts-compiler** (ou encore `XT.compile_to('../RME.rb', '../RME.rvdata2')` si on utilise scripts-compiler mais qu'on veut appeler la compilation depuis un autre script au lieu de ce dernier)

Pour résumer (et m'étendre) un peu d'avantage... chaque **_list.rb** pourrait contenir quatre différents types d'éléments :

- `<instruction_optionnelle>` => instruction pour **scripts-loader** ou **scripts-compiler**
- `nom_du_script_a_charger`
- `nom_du_module_a_charger/` ("module == dossier avec un _list.rb dedans")
- `#commentaire`

Pour le type `<instruction>` pour l'instant j'ai évoqué l'idée de :
- `<compact!>` destiné au **scripts-compiler** => **.rvdata2**
- `<require nom_du_script>`
- `<require nom_du_module/>`

C'est un peu du LESS.rb... sans le ruby. x)

On pourrait renommer **_list.rb** en **foo.bar** je suis ouvert à toutes propositions. Je propose :
une liste de foo :
```
_list, _ls, list, ls, _module, _mod, module, mod
```
une liste de bar :
```
rb, txt, xt, xtl, xtm, exm, aucune
```
Exemple de **foo.bar**: **ls.xtm**, qui se prononcerait "e**XT**ern **M**odule **L**i**S**t"
Autre exemple : "**foo.xtm** == **list.txt** == **module.txt** == **_list.rb**" (**scripts-loader** et **scripts-compiler** prendraient en charge plusieurs types/noms de fichiers selon les préférences de chaque utilisateur, chaque fichier serait écrit de la même manière que **_list.rb**)
