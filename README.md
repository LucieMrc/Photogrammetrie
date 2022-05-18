# Photogrammétrie

Workshop @ Stéréolux

b2bk@hotmail.fr

## 1. Créer un nuage de point avec des images

Choisir un objet à échelle humaine, sans trop de miroir, de verre ou de textures réflechissantes ou trop lisses.

Capturer entre 20 et 200 images, en faisant trois cercles à différentes hauteurs autour de l'objet.

Les rassembler dans un dossier.

### En utilisant RealityCapture

![Screenshot de RealityCapture](./images/capture1.png)

Dans l'onglet `Workflow`, cliquer sur `Import folder` et sélectionner le dossier contenant les images de l'objet.

Tout en haut à gauche de l'interface, sélectionner le mode deux colonnes pour voir d'un coté les images, et de l'autre la scène.

![Screenshot de RealityCapture](./images/capture2.png)

Aligner les images cliquant sur `Align Images` dans l'onglet `Alignement`, ce qui permet de voir apparaître un nuage de points plus ou moins brouillon.

![Screenshot de RealityCapture](./images/capture4.png)

Dans l'onglet `Reconstruction`, cliquer sur `Set Reconstruction Region` puis `Set Region Automatically` pour voir apparaître une boîte autour de l'objet dans le nuage de point.

Grâce aux poignées, adapter la taille de la boîte au plus proche de l'objet.

![Screenshot de RealityCapture](./images/capture3.png)

Toujours dans `Reconstruction`, cliquer sur `Normal Detail` pour créer le modèle 3D.

![Screenshot de RealityCapture](./images/capture5.png)

Une fois que le modèle est généré, cliquer sur `Texture` pour appliquer la texture calculée avec toutes les images.s

![Screenshot de RealityCapture](./images/capture6.png)

Simplifier si besoin puis re appliquer la texture.

Exporter en .xyz en cliquant sur `Mesh` dans la partie `Export`.

![Screenshot de RealityCapture](./images/capture7.png)

### En utilisant Meshroom

Faire glisser les images pour les charger dans Meshroom.

Cliquer sur `Start` pour lancer la procédure.

Le repasser par un logiciel type Blender pour sortir en .xyz.

## 2. Mettre le nuage dans TD

Faire un TOP Point File In, et charger le nuage en .xyz

![Screenshot du TOP Point File In](./images/screen1.png)

Mettre le mode en couleur pour voir la position de chaque point en RGB.

![Screenshot du TOP Point File In](./images/screen2.png)

Faire un TOP Point File Select, et mettre field3, 4 et 5 dans les channels R G B.

![Screenshot du TOP Point Select](./images/screen3.png)

On a des valeurs entre 0 et 255 alors qu'il veux du entre 0 et 1, donc on divise par 255 avec un TOP Math.

![Screenshot du TOP Math](./images/screen4.png)

On crée un null *position* en sortie du Point File In, et un null *color* en sortie du Math.

On ajoute une Camera, une Light et un Geo, ainsi qu'un TOP Render.

![Screenshot du réseau](./images/screen5.png)

En entrant dans le Geo, on supprime le Torus et on met un SOP Add, on coche "On" sur `Add Points` et on laisse en 0, 0, 0.

![Screenshot du SOP Add](./images/screen6.png)

En sortie du Add on met un SOP Convert, dans `Convert to` on met "Particles Per Point", et dans `Particle Type` on met "Render as Point Sprites", et on coche le flag violet et bleu sur le node.

![Screenshot du SOP Convert](./images/screen7.png)

On ressort du Geo, et on active les instances, et on met le null *position* en `Translate OP`, en mettant r g b pour x y z.

![Screenshot du réseau](./images/screen8.png)

Dans l'onglet `Instance 2`, on met le null *color* en `Color Mode` et r g b pour r g b.

Il y a normalement un nuage de points dans le TOP Render.

On crée un MAT Pointsprite, qu'on place dans `Material` dans l'onglet `Render` du Geo. On peux modifier la taille des points avec la variable `Constant Point Scale` dans le MAT Pointsprite.

![Screenshot du MAT Pointsprite](./images/screen9.png)

On va chercher la ArcBallCamera dans la palette, ouvrir le node et copier la camera1 dedans, et le coller dans le projet. 

![Screenshot de la palette](./images/screen10.png)

![Screenshot de la palette](./images/screen11.png)

Supprimer notre camera, et remplacer "cam1" dans le TOP Render par "cam2", celle qu’on a amené.

![Screenshot de la cam](./images/screen12.png)

En sortant de notre projet, dans le node Project1, dans `panel`, cocher le `mouse right`, `mouse left` et `mouse middle`, pour pouvoir modifier la vue avec une souris 3-clics.

![Screenshot du project](./images/screen13.png)

En sortie du TOP Render, mettre un TOP Transform et cocher `Comp over background color` pour mettre un fond en dessous du modèle.

![Screenshot du TOP Render](./images/screen14.png)

## 2. Animer
### L'éclatement en noise

Pour pouvoir modifier les valeurs des points, en couleurs ou en position, on ajoute un TOP Point Transform en sortie du Point File In et en entrée du null *position*.

Ajouter un TOP noise entre le Point Transform et *position* .
Dans l'onglet Output, mettre en mode "input + noise".

Mettre l'`Amplitude` à 0 au début, l'`Offset` à 0, et surtout le `Monochrome` en "Off".
En augmentant l'`Amplitude` on décale donc + ou - les points de toute l'image.

![Screenshot du réseau](./images/screen15.png)

Pour ne pas sélectionner tous les pixels, on peux mettre un threshold sur X, Y ou Z. On ajoute donc un TOP Threshold en sortie du Point Transform.

Pour sélectionner sur X par exemple, on prend le channel Red dans RGB. On peux ajouter du Soften pour faire un dégradé autour de la valeur qu'on met.

![Screenshot du TOP Threshold](./images/screen16.png)

Ajouter un TOP add, et additionner la couleur en sortie du TOP Math et le TOP Threshold, et sortir dans le null *color* pour voir les pixels qu'on sélectionne : ici en blanc au dessus d’un certain X.

![Screenshot du réseau](./images/screen17.png)

Créer un autre TOP Noise, qui sort du Point Transform, avec dans l'onglet `Output` le "noise" seul, et toujours en `Monochrome` "Off".

![Screenshot du TOP Noise](./images/screen18.png)

Mettre un TOP Multiply qui croise ce noise2 et le threshold, pour avoir uniquement les couleurs du noise sur les valeurs du threshold :

![Screenshot du réseau](./images/screen19.png)

On peux ainsi supprimer le TOP add et reconnecter le TOP math à la couleur :

![Screenshot du réseau](./images/screen20.png)

Ajouter un TOP Add entre le Point Transform et le Multiply, afin de mettre du noise uniquement sur les pixels qu’on a sélectionné :

![Screenshot du réseau](./images/screen21.png)

Pour finir, collapse en boite *Noise* le thresh, noise, multiply et add en sélectionnant les 4 nodes et en faisant un clic-droit.

![Screenshot du clic droit](./images/screen22.png)

Dans le node *Noise* :

![Screenshot du réseau](./images/screen23.png)

### La vague

En sortie du node *Noise*, ajouter un Threshold et choisir le channel Green pour sélectionner la position sur Y des pixels.

Sortir le Threshold dans un TOP Level, et jouer dans l’onglet `RGBA` pour teinter tous les pixels qui sortent du Threshold.

![Screenshot du TOP Level](./images/screen24.png)

Sortir du Level dans un Add, dans lequel on entre également la couleur en sortie du Math, et sortir dans le null *color*.

![Screenshot du réseau](./images/screen25.png)

Mettre le Threshold en mode "not equal", pour faire une ligne avec les points autour de notre valeur.

![Screenshot du TOP Threshold](./images/screen26.png)

Pour monter les points sur Y, on met un Level et on fait sortir uniquement le vert, en sortant du même Threshold, et on l’Add au null *position*.

![Screenshot du TOP Level](./images/screen27.png)

On peux aussi faire pareil avec un Channel Mix, en fonction de l'intensité des valeurs de couleurs que l'on cherche.

![Screenshot du TOP Channel Mix](./images/screen28.png)

![Screenshot du réseau](./images/screen29.png)

Pour faire une vague qui se déplace, on fait un CHOP Beat, qui nous fait du 0 à 1 en mode modulo, j’ai mis la période à 12 pour faire un rythme + lent.
On le sort dans un Math pour modifier le range, ici -1, 1, à modifier en fonction de l'échelle de la scène.

![Screenshot du réseau](./images/screen30.png)

Plus le soften du Threshold est grand, plus la bande est large, et plus la valeur de vert du Level est élevée, plus les pixels montent haut.

Pareil, on met tous les noeufs dans une boîte *Wave*.

![Screenshot du clic-droit](./images/screen31.png)

### La limite

En la sortie du node *Wave* et le *position*, ajouter un TOP Limit, et mettre "clamp" en `Limit Type Minimum` et `Maximum`.

![Screenshot du TOP Limit](./images/screen32.png)

En jouant sur les valeurs soit sur la luminance (= toutes les valeurs, en cube autour de la scène), on peux ramener les pixels qui vont trop loin à une limite.

Dans l'onglet `Common` du Limit, on peux décocher des channels de couleurs dans `Channel Mask` pour ne pas influer sur tous les axes de position.

![Screenshot du TOP Limit](./images/screen33.png)

### Le twist

En sortie du Limit, on fait un TOP Threshold sur X (Red), et on crée un nouveau TOP Point Transform, où on rotate de 1 sur X.

On rentre dans la seconde entrée avec le Threshold, qui permet de faire un masque, et dans la première entrée avec le Limit.

![Screenshot du réseau](./images/screen34.png)

On crée un TOP Feedback, dont on rentre avec le Limit et on sort dans le Point Transform, et son Target TOP est le Point Transform.

![Screenshot du TOP Feedback](./images/screen35.png)

On peux collapse les trois dans un node *Twist*.

### La souris

Créer un CHOP Mouse In, et ajouter le bouton gauche qu’on appelle "lb".

![Screenshot du CHOP Mouse In](./images/screen36.png)

Créer un TOP Constant noir (pour l'instant).

Sortir un Null *position* de *wave*, et le rentrer dans le Constant juste pour lui donner la bonne taille.

![Screenshot du réseau](./images/screen37.png)

Changer la couleur du Constant pour faire une couleur, ici 0, 1, 0, mais c’est arbitraire.

![Screenshot du TOP Constant](./images/screen38.png)

Faire un TOP Substract et rentrer le Constant en haut et le null *position* en bas.

![Screenshot du réseau](./images/screen39.png)

Faire un Math, mettre le Substract en entrée, et mettre `Combine Channels` en mode "Length".

![Screenshot du TOP Math](./images/screen40.png)

Changer le `Pixel Format` en "32 bit mono" : on a ainsi la distance en noir et blanc entre chaque pixel de l’image (x, y, z) et la coordonnée du Constant (0, 1, 0).

![Screenshot du TOP Math](./images/screen41.png)

Rajouter un Threshold en sortie du Math, et mettre le comparateur en "Greater".

![Screenshot du TOP Threshold](./images/screen42.png)

Faire un Add et ajouter ça à la couleur pour voir apparaître le rond de distance de la coordonnée constant en blanc (ici j’ai changé la couleur du Constant pour trouver un truc au milieu de la moto).

![Screenshot du réseau](./images/screen43.png)

Faire glisser le "tx" de la souris dans le rouge et le "ty" dans le bleu, et faire *10 pour que ça bouge d'avantage si besoin, en fonction de l'échelle du nuage de points.

![Screenshot du TOP Constant](./images/screen44.png)

Après le Threshold, ajouter un Level qu’on remet en "32-bit RGBA" dans le `Pixel Format`.

![Screenshot du TOP Level](./images/screen45.png)

Changer les channels couleurs dans l'onglet `RGBA` et le refaire rentrer dans le Add.

![Screenshot du TOP Level](./images/screen46.png)

Créer en sortie du Level2 un Noise, en `Output`, sélectionner juste "Noise", et on le rentre dans le TOP Multiply avec le Threshold.

![Screenshot du réseau](./images/screen47.png)

On met le Noise pas trop fort, et on Add ça à la *position*.

![Screenshot du réseau](./images/screen48.png)

Après on ajoute un Constant pour faire monter sur Y ici (donc un peu de vert), et pareil on le met sur *position*.

![Screenshot du réseau](./images/screen49.png)

Si on veux mettre le particle machin, on fait rentrer *position* et *couleur*, et sortir dans *position* et *couleur*.

Pour sortir une video, en sortie du out mettre un TOP Movie File Out.
On décoche "Realtime" en haut à gauche de l'interface, on fait "Record" et au bout d’un moment on arrête le Record.

Cocher autorotate dans archball dans la camera pour que la camera tourne autour.
















































