Ce dossier contient l'essentiel du travail de développement réalisé lors de mon stage de 4 mois au sein du cabinet d'études sur l'énergie et le climat [__Enerdata__](https://www.enerdata.fr/).

## Contexte du stage

L'une des principales activités d'Enerdata consiste à développer des bases de données couvrant divers aspects des questions énergétiques et climatiques. Ces bases font à la fois office d'outils de travail pour les experts énergéticiens de l'entreprise, de matériau de base pour les modèles prospectifs et de produits vendus sous forme d'accès par abonnement à des acteurs privés ou institutionnels : grandes entreprises, cabinets de conseil, universités, etc.

Jusqu'en 2021, le traitement des données de leur collecte à leur mise en production reposait essentiellement sur deux piliers : 
- le SGBD Oracle pour le stockage et le versioning des données d'un bout à l'autre du process ;
- le PL/SQL, qui est un langage propriétaire d'Oracle pour la réalisation de traitements de données au sein même de la base relationnnelle.

Ce _data process_ présentait plusieurs limites :
- Maintenance : la technicité du PL/SQL et la nécessité de manipuler de multiples fichiers pour l'exécution du traitement impliquait de répartir le travail de mise à jour entre une équipe métier chargée de définir et de vérifier le contenu de la base et une équipe développement chargée de modifier le code, au prix d'importants coûts organisationnels ;
- Vitesse : malgré la précompilation du code sous Oracle, le temps de calcul pour les plus grandes bases pouvait s'étaler sur pluseurs jours ;
- Coût : Oracle et par extension le PL/SQL sont des solutions payantes.

Enerdata a donc décidé courant 2021 d'entamer la transition vers une nouvelle architecture de traitement de données fondée sur un ___datalake_ HDFS__ et sur les outils de __l'écosystème Python__ (Numpy, Pandas, notebooks Jupyter...) afin de surmonter les problèmes mentionnés ci-dessus :
- Maintenance : le langage Python est réputé pour sa simplicité, sa polyvalence, et dispose d'une documentation abondante pour aider à son utilisation. De son côté, le datalake permet de stocker des fichiers bruts, sans structure, et de les importer/exporter avec une facilité accrue par rapport à Oracle ;
- Vitesse : le calcul vectorisé sous __Numpy__ et le stockage des données à traiter sur la mémoire vive grâce aux tableaux de données __Pandas__ (là où le PL/SQL doit accéder au disque dur et prend donc plus de temps) permettent d'atteindre des performances bien supérieures en termes de temps de calcul ;
- Coût : ces outils sont sous licence libre et donc utilisables gratuitement.

## Présentation de la base Odyssée et contenu du stage

A mon arrivée chez Enerdata en mai 2022, je me suis vu confier l'implémentation du _data process V3_ au niveau de la base de données __Odyssée__ sous la supervision conjointe de __Laura Sudries__, cheffe de projet sur Odyssée, et de __Julien Leroux__, _data architect_ de l'entreprise. Ce dernier m'a notamment introduit aux outils de développement utilisés : _template_ de base pour le notebook principal, Visual Studio et GitLab.

[Odyssée](https://www.enerdata.fr/research/demande-energetique-mondiale-enerdemand.html) est une base de données consacrée au suivi de la demande et de l'efficacité énergétique en Europe. Elle s'inscrit dans un [projet plus large](https://www.odyssee-mure.eu/project.html) de création d'un outil d'évaluation des politiques d'efficacité énergétique supervisé par l'ADEME et co-financé par la Commission européenne. <br>
Du fait de cette composante publique européenne, l'une des particularités d'Odyssée était de collecter ses données brutes directement auprès des ministères et agences de l'énergie d'une trentaine de pays partenaires du projet.

La phase annuelle de collecte nécessitait cependant de nombreux échanges avec les partenaires ministériaux afin de remplir les templates Excel conçus pour accueillir les données de chaque pays. Pour simplifier ces démarches en parallèle du changement de process, l'équipe en charge d'Odyssée a pris le parti de changer la source de 70% de ses données en les récupérant d'Eurostat, directement ou via une autre base d'Enerdata, _Global Energy & Demand_ (GED), qui prélevait déjà une partie de ses données sur Eurostat.

Mon travail a donc consisté dans un second temps à mettre en place les outils permettant de remplacer les données de sources nationales par les données Eurostat grâce à des fichiers de mapping produits par mon équipe d'accueil. Il n'a cependant pas abouti concernant les données GED à cause d'un problème de mappings incohérents entre les équipes en charge des deux bases, problème qui n'a pas pu être corrigé avant mon départ.

## Résumé du travail réalisé

1) Traduction des appels aux procédures PL/SQL contenus dans les fichiers d'exécution pour les adapter à la syntaxe des fonctions contenues dans le notebook de traitement (itération sur fichier + RegEx).
2) En parallèle, modification et création d'une quinzaine de fonctions (certaines étaient fournies de base) remplaçant la centaine de procédures codées en PL/SQL en suivant la contrainte "une fonction = plusieurs usages".
3) Comparaison des résultats obtenus entre l'ancien et le nouveau process et correction du notebook Odyssée.
4) Mapping et remplacement des données pays par leur équivalent Eurostat ou GED. 
6) Création d'un launcher permettant de traiter les séries de plusieurs pays en une seule exécution et de distribuer les calculs.
7) Mise au point d'un (modeste) outil de comparaison visuelle des séries destiné aux experts de l'équipe en charge d'Odyssée.

## Description des notebooks

[_Odyssée_](https://github.com/Tim-De-Decker/Stage_Enerdata/blob/main/Odyss%C3%A9e.ipynb)

Le notebook Odyssée est dédié au calcul des séries placées dans la base de production à partir des données collectées auprès des partenaires du projet Odyssée-Mure, sur Eurostat ou dans les bases constitutives de GED. <br>
Les données sources y sont stockées dans un dataframe (un tableau de données) nommé resOdy qui est mis à jour à chaque nouvelle ligne de calcul. <br>
Le notebook Odyssée inclut une quinzaine de fonctions documentées ainsi qu’une partie Calculs où chaque série de la base de production est calculée via un appel de fonction. Ces appels fournissent en même temps un aperçu de la formule appliquée pour obtenir la série finale. <br>
Exécuter l’ensemble du notebook permet de calculer les séries d’un unique pays en un peu moins de deux minutes.

[_Traducteurs_](https://github.com/Tim-De-Decker/Stage_Enerdata/blob/main/Odyss%C3%A9e.ipynb)

Ce notebook de travail a servi à traduire les fichiers d'appel des procédures PL/SQL pour les adapter aux nouvelles fonctions créées sous Python. Il contient une boucle de traduction basée sur des RegEx par fichier initial ainsi qu'une boucle chargée de nettoyer les fichiers traduits. Les fichiers de base et leurs versions traduites se trouvent dans le dossier [FichiersSQLTrad](https://github.com/Tim-De-Decker/Stage_Enerdata/tree/main/FichiersSQLTrad).

_[Calcul pays](https://github.com/Tim-De-Decker/Stage_Enerdata/blob/main/Calcul%20pays.ipynb) et [Launcher Odyssée](https://github.com/Tim-De-Decker/Stage_Enerdata/blob/main/Launcher%20Odyssee.ipynb)_

L’intérêt de ces deux notebooks est de permettre le lancement des calculs de plusieurs pays en parallèle sur les différents cœurs du processeur. Il suffit pour cela de paramétrer __Launcher Odyssée__ (liste des pays souhaités + dates de début et de fin des données) puis de l’exécuter dans son ensemble : le launcher va alors passer par __Calcul pays__ pour créer et lancer en simultané des versions d’Odyssée paramétrées pour chacun des pays de la liste renseignée.<br>
Le processeur actuel comporte 4 cœurs : autrement dit, le launcher permet grossièrement de diviser par 4 le temps de calcul par pays obtenu grâce au notebook classique. <br>
Le calcul de toutes les séries de l’ensemble des 33 pays référencés par Odyssée prend ainsi approximativement 18 minutes contre près d'une demi-journée avec l'ancien process.

_[Import GED](https://github.com/Tim-De-Decker/Stage_Enerdata/blob/main/Import%20GED.ipynb) et [Import Eurostat](https://github.com/Tim-De-Decker/Stage_Enerdata/blob/main/Import%20Eurostat.ipynb)_

Comme leurs noms l’indiquent, les notebooks __Import GED__ et __Import Eurostat__ servent à l’importation des nouvelles données collectées sur la base des fichiers de mapping Excel contenus dans le dossier [Mapping](https://github.com/Tim-De-Decker/Stage_Enerdata/tree/main/Mapping) et des fichiers tsv extraits d'Eurostat contenus dans le dossier [tsvEurostat](https://github.com/Tim-De-Decker/Stage_Enerdata/tree/main/tsvEurostat). <br>
Après exécution, les données sont stockées dans deux fichiers csv en vue de leur future utilisation : donneesGED.csv et donneesEurostat.csv.

_[ComPy](https://github.com/Tim-De-Decker/Stage_Enerdata/blob/main/ComPy.ipynb)_

__ComPy__ est un petit outil servant à la comparaison graphique des séries obtenues grâce aux données Oracle et de celles découlant des nouvelles données Eurostat et GED. <br>
L’outil propose deux types de comparaison : visualisation des séries-clés définies pour chaque secteur et visualisation libre via une liste renseignée par l’utilisateur.

_[scriptRawData](https://github.com/Tim-De-Decker/Stage_Enerdata/blob/main/scriptRawData.ipynb)_

Ce notebook relativement réduit permet d’exporter le fichier rawdata au format xlsx. Ce fichier récupère une sélection des nouvelles données importées afin que celles-ci soient réinjectées dans le template de collecte où sont réalisés certains calculs en amont du process.
