#w8-s5

Maitenant qu'on a vu les mécanismes de
base du langage pour créer des objets
coroutines, nous allons creuser un peu
la notion d'objet awaitable.

---------- slide protocole awaitable

Alors c'est quoi un objet awaitable ?

En fait, pour faire une analogie, on a
vu la notion d'itérable qui correspond
aux objets sur lesquels on peut faire un
for; et les context managers qui peuvent
faire l'objet d'un with;

eh bien c'est pareil ici, le concept de
awaitable décrit la classe des objets
sur lesquels on peut faire un await.

Les awaitables sont donc une abstraction
des coroutines, ou dit dans l'autre
sens, les objets coroutines sont un
cas particuler d'objets awaitables, mais
on peut en définir d'autres, via la
méthode spéciale await.

========== __await__ revoie un itérateur

Cette méthode magique __await__ doit
renvoyer un itérateur; comme avec les
itérateurs standard, il est tentant
d'implémenter un itérateur avec une
fonction génératrice; donc je vous
montre un tout premier exemple de classe
awaitable, qui implémente __await__
comme un itérateur à un coup, qui yield
une fois la constante 10 et puis
termine.

---------- fragment
On va voir comment je peux 
utiliser un objet de ce genre, en dehors
de la librairie asyncio completement;
mon objectif ici c'est de vous donner
une idée de comment la boucle asyncio
fait son travail.

Bon pour pouvoir faire await, vous vous
souvenez je ne peux pas le faire depuis
une fonction normale, je crée donc une
coroutine qui await mon objet awaitable

========== fragment

maintenant je crée un objet coroutine,
et je vais vous montrer qu'en fait

========== 

je peux "faire avancer" cette fonction
coroutine en envoyant à l'objet
coroutine la méthode send.

Pour l'instant je le fais une seule
fois, mais

========== slide
je prends un deuxième exemple à peine
plus compliqué, sauf que la méthode
await, en fait le générateur qui
implémente la méthode await, est à deux
coups.

Si je refais exactement comme à
l'instant, voici ce qui va se passer

---------- fragment (1er send)

la première fois que j'envoie send à la
coroutine, elle va jusqu'au premier
yield, puis elle se suspend, exactement
comme un itérateur dans un for.

---------- fragment (2eme send)

la deuxième fois elle avance encore un
cran jusqu'au deuxième yield, jusque là
tout va bien

---------- fragment (3eme send)

et par contre la troisieme fois,
exactement comme si était dans un for,
la methode send reçoit une exception
StopIteration, avec comme valeur la
valeur de retour final de l'awaitable.

---------- slide plusieurs travaux 

Donc vous voyez l'idée, pour implémenter
une boucle, on doit gérer un certain
nombre de choses à faire en même temps,
et avec ce mécanisme de send() on peut en
sélectionner une et lui dire de
continuer jusqu'au prochain yield.

--- dérouler le slide

ici par exemple je prends deux
coroutines à deux coups, comme à
l'instant, je peux les faire avancer
chacune deux fois dans l'ordre que je
veux, ici un coup sur deux.


---------- slide pile, await et yield

Je vais finir cette vidéo avec un
exemple un tout petit peu plus élaboré,
c'est surtout pour bien vous montrer la
différence entre await et yield.

Dans ce que j'ai dit jusqu'ici, surtout
en début de semaine, vous pourriez avoir
retenu que c'est avec await que l'on
rend explicitement le contrôle à la
boucle.

En fait c'est vrai en première
approximation, mais si on regarde dans
le détail il est plus précis de dire que
c'est yield qui fait ce travail. On
vient déjà de le voir mais dans des
exemples avec un seul await, donc je
vais finir cette vidéo avec un exemple
où il y a quelques appels de coroutines
imbriqués, pour que ça soit très clair.

---------- fragment awaitablevalue

je me donne une classe de awaitable très
simple, comme dans mon tout premier
exemple la méthode await ne fait qu'un
seul yield. Bon ici j'ai choisi de
retourner la valeur qu'on utilise pour
créer l'objet, et aussi j'ai mis un
compteur de classe, que j'utilise pour
yielder, au fur et à mesure je vais
yielder le string 'y 1', 'y 2' etc

c'est pour qu'on s'y retrouve

---------- fragment async def w4()
maintenant je vais créer une coroutine
w4, qui appelle avec await une coroutine
w3(), qui appelle une coroutine w2(),
qui appelle 2 fois w1()

Ce que je veux vous montrer surtout ici,
c'est que les await en soit ne
provoquent pas une commutation, mais
seulement les yield.

Donc regardons d'abord comment ça se
présente si j'envoie plusieurs fois
send() à ma coroutine

la première fois je reçois 'y 1' qui
correspond au premier yield, la seconde
je reçois 'y 2', et la troisième à
nouveau je reçois l'exception
StopIteration, avec comme valeur la
valeur de retour de la coroutine.

========== slide animation

Je vous montre rapidement une animation
pour bien comprendre comment ça se passe
en termes de pile notamment

je commence avec un objet coroutine qui
est vierge, lorsque j'envoie le premier
send l'exécution commence; je mets la
coroutine en rouge tant qu'elle garde la
main.
donc on empile la fonction w4, on arrive
à await, du coup on empile w3, puis w2
et enfin w1

là quand le awaitable w1 arrive au
yield, alors le premier send retourne
exactement ce lui a passé le yield, et là pour la
première fois on peut récupérer la main
puisque send() a retourné.

du coup l'on peut faire autre chose

lorsqu'on envoie le deuxième send, la
pile est restaurée exactement dans
l'état où elle était - à nouveau comme
un générateur traditionnel - w1 se
poursuit après le yield, retourne à w2
la valeur 1, se fait dépiler, puis dans le
code de w2 il y a un second appel à w1()
et donc on recommence à l'empiler,
tout ça c'est toujours dans le deuxième
appel à send(), puis w1 arrive à yield,
et là comme tout à l'heure: la valeur de
retour de send vient exactement du
yield, et surtout on remet la coroutine
au freezer jusqu'au troisième appel à
send

Là on réactive la coroutine, w1 se
termine, on dépile tous les appels comme
si c'était du code normal, et comme w4
est finie et qu'elle retourne 4, send()
ne reçoit pas de valeur de retour, mais
l'exception StopIteration est levée avec
4 comme valeur


========== fermer animation

---------- slide conclusion

Bon voilà, vous voyez la mécanique
générale, tout ce paradigme de
programmation asynchrone repose beaucoup
sur les générateurs, qui sont présents
dans le langage depuis très longtemps.

Je résume ce qu'on a vu cette fois ci

---------- fragment
on a vu comment se définir notre propre
classe de awaitable

---------- fragment
on a vu comment la méthode send permet
à la boucle de faire avancer les
coroutines

---------- fragment
jusqu'au prochain endroit dans
l'exécution où on rencontre un yield


On verra dans les compléments qu'avec
send et yield, il y a en fait une
communication dans les deux sens entre
la boucle et les coroutines.


J'attire votre attention sur le fait que
je n'ai pas importé asyncio dans ce
notebook, tout ça ce sont des mécanismes
natifs du langage, et bien entendu la
boucle d'événements asyncio utilise tout
ça, sauf qu'elle ajoute de l'intelligence par
rapport aux entrées sorties pour faire
les bons choix au bon moment.

À bientôt

