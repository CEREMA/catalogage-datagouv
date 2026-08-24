# catalogage des données de mobilité de data gouv

ces notebooks implémentent des étapes de recherche de jeux de données open data.

l'objectif final est de pouvoir créer un portail des données de mobilité, complémentaire de transport data gouv, et de logistique data gouv, sur le périmètre des données pour l'analyse des mobilités.

l'objectif technique à court terme est de pouvoir classer les données de data.gouv en collections de données par thème métier 
(stationnement, voirie, transport public, trafic routier, logistique et fret, vélo, marche, accessibilité, mobilité partagée...) et par sous-thème dans chaque métier.

l'approche est de mettre en place un pipeline de traitement avec les étapes suivantes:
0) sélection des jeux de données du domaine mobilité/transport
pour chaque thème
1) sélection des jeux de données pertinents pour le thème
2) étiquetage des jeux de données par sous-thème
3) tests de la qualité des données pour quelques critères simples (date de mise à jour, etc.)
4) publication des collections de données par thème (dans un tableau Grist et/ou en tant que collection dans ecologie.data.gouv)
5) envoi de messages aux producteurs de données (retour sur la qualité des méta-données)

le catalogue de data.gouv.fr contient environ 75k datasets

#### inspiration de flowdatagouv
https://github.com/FLI-GCT/FlowDataGouv est un démonstrateur développé en mars 2026 par Guillaume Clément à l'occasion de la publication du serveur MCP de data.gouv.fr
L'idée est d'utiliser le serveur MCP pour classer, étiqueter et nettoyer les 75k datasets de data.gouv et proposer un site alternatif à data.gouv pour accéder. Le site a été fermé après 1 mois (PoC).
Dans ce projet (dont le code a été généré par claude), les principales données des datasets de data.gouv sont récupérées via l'API et mises dans un dataframe (code en typescript : src/lib/sync/catalog.ts), et enrichies par Mistral AI (catégorie, sous-catégorie, zone géographique, résumé, score qualité — voir enrichBatchMistral).
Un contrôle qualité des datasets et ressources est également effectué, avec calcul d'indicateurs de qualité (en python).
Ce projet est vraiment très proche de ce que nous voulons faire (uniquement pour le domaine mobilité, et en python).

## 0) sélection des données mobilité/transport
peut être fait par un prompt, l'objectif est qu'il y ait peu ou pas de faux négatifs, pas forcément d'éliminer tous les faux positifs

## 1) sélection des jeux de données par thème
Plusieurs approches sont possibles : soit de faire n sélections indépendamment pour chaque thème, soit de faire toutes les requêtes directement dans une même requête sur le catalogue complet data.gouv. 
Un même dataset peut être dans plusieurs thème (ex stationnement vélo)

### prompt
Un prompt peut permettre d'avoir rapidement des résultats et de valider une liste. 
Comme le code, les prompts doivent être documentés, par ex. publiés sur github sous forme de textes, en précisant quel modèle est interrogé.

Le thème test est :  les données de trafic routier. la liste des datasets qui semble valide comprend 135 datasets.

### recherche textuelle
Comme le prompt n'est pas forcément reproductible, on souhaite trouver un autre mode de requête par simple recherche textuelle de présence de mots-clés dans le titre et la description de chaque dataset
une approche intermédiaire serait de faire appel à une lib de modèle de langage au lieu de prompter un IA gen.
La recherche peut être simplement de chercher si une une liste de mots-clés est présente dans le titre, dans la description, aussi (ou d'abord?) dans les mots-clés du dataset, voir éventuellement dans titre et description des ressources (fichiers etc.) attachées au dataset.

### requête par modèle de langage
le principe est le suivant :
- chaque dataset est décrit par un vecteur (embedding) dans le modèle de langage
- chaque requête thématique est décrit par un vecteur (embedding) dans le modèle de langage
La proximité entre dataset et thème ou sous-thème est évaluée comme le produit (cosinus) entre les 2 vecteurs

## 2) étiquetage des datasets par sous-thème
Les pages https://cerema.github.io/mobscidat/ permettent de proposer une 1ere liste de sous-thèmes

L'étiquetage peut être fait en même temps que la recherche par thème (que ce soit par prompt, recherche textuelle, ou par modèle de langage).

