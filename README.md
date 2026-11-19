Bonjour Khalil,

J'ai repris les vérifications en PREX sur le flux **Résultat de cession FIFO** et je pense avoir compris l'origine du gros écart de volumétrie.

**1. Comparaison des volumes en PREX**

J'ai retrouvé les volumes suivants :

- Table SI3 `delta` : **3 042 lignes**
- Table SI3 `delta_histo` : **783 347 lignes**
- Table DF6 `mrv_resultat_cession_fifo_quot` : **786 388 lignes**

En fait, on comparait le DELTA courant avec le MAS qui contient les données du chargement initial et des chargements suivants.

**[Capture Hue 1 : les COUNT(*) des trois tables en PREX]**

**2. Vérification des chargements dans le MAS**

J'ai ensuite regardé la colonne `ldts` du MAS pour comprendre les chargements effectués.

On retrouve deux chargements le 18/11 :

- À **17:52:29** : 783 347 lignes, ce qui correspond au volume du `delta_histo` utilisé pour l'initialisation.
- À **18:19:06** : 3 041 lignes, ce qui correspond presque au volume du DELTA courant.

Donc le MAS contient bien les données de l'INIT, auxquelles s'ajoute un deuxième chargement.

783 347 + 3 041 = **786 388 lignes**.

**[Capture Hue 2 : requête GROUP BY ldts avec les deux chargements]**

**3. Pourquoi on ne retrouve pas cet écart en RECETTE**

J'ai refait les mêmes vérifications en RECETTE.

On retrouve **901 352 lignes** dans les trois tables : `delta`, `delta_histo` et MAS.

En regardant le `ldts`, on voit un seul chargement dans le MAS, datant du 03/11.

Donc en RECETTE, on voit uniquement le chargement initial, contrairement à la PREX où un deuxième chargement a eu lieu.

C'est ce qui explique pourquoi la comparaison donnait le même résultat en RECETTE, mais pas en PREX.

**[Capture Hue 3 : les trois COUNT(*) en RECETTE]**

**[Capture Hue 4 : GROUP BY ldts du MAS en RECETTE, avec un seul chargement]**

**4. Conclusion**

Le gros écart entre les 3 000 et les 700 000 lignes s'explique donc principalement par le fait qu'on compare le DELTA courant avec un MAS qui cumule plusieurs chargements.

Il reste un petit écart d'une ligne entre le DELTA (3 042) et le deuxième chargement du MAS (3 041). Je ne l'ai pas encore investigué.

J'ai également vérifié les autres MAS Résultat de cession (quotidien et mensuel), leurs volumes correspondent bien aux tables `delta_histo` SI3.

**[Capture Hue 5 : les COUNT(*) des autres MAS et leurs sources SI3]**
