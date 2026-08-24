# catalogage des données de mobilité de data gouv

ces notebooks testent

l'objectif final est de pouvoir créer un portail des données de mobilité, complémentaire de transport data gouv, et de logistique data gouv, sur le périmètre des données pour l'analyse des mobilités.

l'objectif technique à court terme est de pouvoir classer les données de data.gouv en collections de données par thème métier 
(stationnement, voirie, transport public, trafic routier, logistique et fret, vélo, marche, accessibilité, mobilité partagée...) et par sous-thème dans chaque métier.
l'approche est de mettre en place un pipeline de traitement avec les étapes suivantes
0) sélection des jeux de données du domaine mobilité/transport
pour chaque thème
1) sélection des jeux de données pertinents pour le thème
2) étiquetage des jeux de données par sous-thème
3) tests de la qualité des données pour quelques critères simples (date de mise à jour, etc.)
4) publication des collections de données par thème (dans un tableau Grist et/ou en tant que collection dans ecologie.data.gouv)
5) envoi de messages aux producteurs de données (retour sur la qualité des méta-données)

le catalogue de data.gouv.fr contient environ 75k datasets

## 0) sélection des données mobilité/transport
peut être fait par un prompt, l'objectif est qu'il y ait peu ou pas de faux négatifs, pas forcément d'éliminer tous les faux positifs

## 1) sélection des jeux de données par thème
plusieurs approches sont possible : soit de faire n sélections indépendamment pour chaque thème, soit de faire toutes les requêtes directement 
un prompt peut permettre d'avoir rapidement des résultats et de valider une liste
comme le code, les prompts doivent être documentés, par ex. publiés sur github sous forme de textes, en précisant quel modèle est interrogé.
le thème test est les données de trafic routier. la liste des datasets qui semble valide comprend 135 datasets.
comme le prompt n'est pas forcément reproductible, on souhaite trouver un autre mode de requête par simple recherche textuelle de présence de mots-clés dans le titre et la description de chaque dataset
une approche intermédiaire serait de faire appel à une lib de modèle de langage au lieu de prompter un IA gen.
un même dataset peut être dans plusieurs thème (ex stationnement vélo)

## 2) étiquetage des datasets par sous-thème
les pages https://cerema.github.io/mobscidat/ permettent de proposer une 1ere liste de sous-thèmes
