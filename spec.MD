# cahier des charges d'un catalogue thématique des données de data.gouv.fr

Nous voulons créer un pipeline (en python) permettant d'extraire et mettre à jour les données , réaliser les étapes suivantes :
0. sélection des données sur le domaine de la mobilité
1. création de listes des datasets sur chaque thème
2. étiquetage des datasets sur des sous-thèmes pour chaque thème
3. ajout d'attributs supplémentaires attachés aux datasets permettant de tester , par exemple la date de dernière de mise à jour
4. publication des résultats dans des tableaux Grist
5. notification des producteurs de données de la publication et suggestions d'amélioration sur les données (ajout de mot-clé par exemple)

La définition du domaine de la mobilité est défini à partir d'une liste de "métier" ou thèmes. Pour chaque thème, les sous-thèmes ont définis à partir de chaque page présentant les types de données par métier : https://cerema.github.io/mobscidat/ 

La liste initiale est : stationnement, voirie, transport public (et rail), vélo, marche et accessibilité, trafic routier, fret et logistique, mobilité partagée (covoiturage, autopartage et vélo et trottinettes en libre-service), enquêtes, données de référence (parc, comptes transport), données environnementales (émissions de GES, bruit, qualité de l'air), accidentologie, plus un thème "autres" qu'on pourra affiner au fil de l'eau.

Besoin : il doit être possible de compléter la définition du domaine Mobilité (ajouter / corriger des thèmes)

Besoin : il doit être possible de compléter la définition des thèmes et sous-thèmes. Les datasets qui ne rentrent pas dans un sous-thème seront dans le sous-thème "autres". 

Besoin Il doit être possible de compléter la liste des sous-thèmes / types de données de chaque thème.

Un dataset peut être dans plusieurs thèmes, et dans plusieurs sous-thèmes d'un même thème. Notamment un dataset comprend parfois des ressources (données en général) correspondant à des types de données différents (ex. description des capteurs de trafic, et données mesurées).

Besoin : Il doit être possible de mettre à jour manuellement la liste de chaque thème:
- ajout à la main de datasets de data.gouv ou non publiés dans data.gouv
- suppression de datasets mal ("faux positifs")

Besoin : Il doit être possible de valider "à la main" qu'un dataset est bien dans un thème ou un sous-thème. 

L'intérêt est qu'une fois validé un dataset pour un thème ou sous-thème, comme l'id du dataset est pérenne, on peut conserver ce dataset pour les mises à jour du catalogue.

Comme pour les datasets, il est prévu dans un 2ème temps de pouvoir classer, regrouper par thème et sous-thème les autres données du catalogue datagouv : ressources, organisations, API, mots-clés... https://www.data.gouv.fr/datasets/catalogue-des-donnees-de-data-gouv-fr 

Pour la sélection des données du domaine mobilité, puis pour le regroupement des datasets, plusieurs méthodes sont envisagées et vont être testées :
1) prompt à un LLM ; l'inconvénient est qu'on n'a alors peu d'explicabilité de la classification réalisée, et aussi que cela demande un travail, à moins qu'on utilise l'API du LLM (ce qui n'est pas a priori notre orientation car nous n'avons pas de licence payante)
2) recherche basique de mots présents dans les attributs du dataset : titre, description, tags, voire dans les titres et descriptions des ressources associées au dataset. Cette méthode est la plus simple et la plus rapide, l'inconvénient est qu'elle est très fruste donc a priori retourne une classification médiocre.
3) recherche "évoluée" de proximité entre les les attributs du dataset et la description textuelle (ou liste de mots-clés) du thème ou du sous-thème. Cette recherche "évoluée" utiliserait une lib de modèle de langage open source (comme https://github.com/huggingface/sentence-transformers) est un peu intermédiaire entre les 2 précédents, mais reste à valider également et peut exiger pas mal de ressources pour l'inférence.
