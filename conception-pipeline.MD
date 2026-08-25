Nous voulons créer un pipeline (en python) permettant d'extraire et mettre à jour les données , réaliser les étapes suivantes : 0. sélection des données sur le domaine de la mobilité

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

* ajout à la main de datasets de data.gouv ou non publiés dans data.gouv
* suppression de datasets mal ("faux positifs")

Besoin : Il doit être possible de valider "à la main" qu'un dataset est bien dans un thème ou un sous-thème.
L'intérêt est qu'une fois validé un dataset pour un thème ou sous-thème, comme l'id du dataset est pérenne, on peut conserver ce dataset pour les mises à jour du catalogue.
Comme pour les datasets, il est prévu dans un 2ème temps de pouvoir classer, regrouper par thème et sous-thème les autres données du catalogue datagouv : ressources, organisations, API, mots-clés... https://www.data.gouv.fr/datasets/catalogue-des-donnees-de-data-gouv-fr
Pour la sélection des données du domaine mobilité, puis pour le regroupement des datasets, plusieurs méthodes sont envisagées et vont être testées :

1. prompt à un LLM ; l'inconvénient est qu'on n'a alors peu d'explicabilité de la classification réalisée, et aussi que cela demande un travail, à moins qu'on utilise l'API du LLM (ce qui n'est pas a priori notre orientation car nous n'avons pas de licence payante)
2. recherche basique de mots présents dans les attributs du dataset : titre, description, tags, voire dans les titres et descriptions des ressources associées au dataset. Cette méthode est la plus simple et la plus rapide, l'inconvénient est qu'elle est très fruste donc a priori retourne une classification médiocre.
3. recherche "évoluée" de proximité entre les les attributs du dataset et la description textuelle (ou liste de mots-clés) du thème ou du sous-thème. Cette recherche "évoluée" utiliserait une lib de modèle de langage open source (comme https://github.com/huggingface/sentence-transformers) est un peu intermédiaire entre les 2 précédents, mais reste à valider également et peut exiger pas mal de ressources pour l'inférence.


Pour l'instant nous avons testé quelques requêtes (prompt LLM, recherche textuelle), sur un  thème test : les données de trafic routier.
Nous avons aussi fait des tests par recherche textuelle et modèle de langage à partir d'une sélection des datasets de datagouv, sur ce repo : https://github.com/CEREMA/catalogage-datagouv/ 

Néanmoins nous avons du mal à avancer, les tests ne sont pas vraiment satisfaisants, la classification par sous-thème a des faux négatifs et des faux positifs. On a l'impression qu'il faudrait quand même faire une 1ere version avec un pipeline fonctionnel qui classifie même mal l'ensemble des datasets puis qu'on améliore à la main la classification puis qu'on itère.
Quel est ton avis ?
Comment arriver à notre catalogue de données de mobilité, concrètement ?

Claude Sonnet 5

## Ton instinct est le bon

Oui : construis d'abord un **pipeline de bout en bout, même médiocre**, plutôt que de chercher la classification parfaite avant de brancher Grist et la boucle de validation. Deux raisons concrètes :

1. Tant que tu n'as pas de **corpus validé à la main**, tu n'as aucun moyen objectif de comparer méthode 2 (mots-clés) vs méthode 3 (embeddings) vs LLM. Le premier passage moyen, une fois corrigé à la main, *devient* ton jeu de test.
2. Le vrai risque n'est pas la qualité initiale de la classification, c'est de **perdre le travail de validation manuelle** à chaque réexécution. Une fois ce point réglé architecturalement, la qualité de l'auto-classif devient un problème secondaire qu'on améliore en continu.

## Le point d'architecture le plus important : séparer "classification" et "vérité validée"

Ne stocke jamais le résultat de l'algorithme comme la donnée finale. Il te faut une table pivot persistante, indépendante du run du pipeline :

```
(dataset_id, theme, sous_theme) → statut, source, score, date_validation, validateur
```

avec `statut` ∈ {`candidat_auto`, `valide`, `rejete`, `ajout_manuel`}.

Règle de fusion à chaque exécution du pipeline :
- si le triplet existe déjà avec `valide`/`rejete`/`ajout_manuel` → **on ne touche à rien**, quel que soit ce que dit l'algo ;
- sinon → on écrit/actualise `candidat_auto` avec le nouveau score.

Cette table peut très bien **être une table Grist elle-même** : Grist devient à la fois l'outil de publication *et* l'outil de validation (colonnes cochables, filtres, vues par thème). Un script de sync relit l'état Grist avant de republier, donc les gestionnaires du catalogue travaillent directement dans l'outil final, pas dans un outil de review séparé.

## Sur la méthode de classification : ne pas choisir une seule méthode, les faire jouer des rôles différents

- **Méthode 2 (mots-clés)** → utilise-la pour la **sélection large du domaine mobilité (étape 0)**, en visant le rappel plutôt que la précision (mieux vaut quelques faux positifs à éliminer à la main que rater des datasets). C'est explicable, rapide, et suffisant à ce niveau.
- **Méthode 3 (sentence-transformers)** → utilise-la pour le **score de proximité au sous-thème**, avec un modèle multilingue/français (`paraphrase-multilingual-mpnet-base-v2` ou `sentence-camembert-*`). Elle ne décide pas seule : elle sert à **trier/prioriser** la file de validation humaine (les scores élevés remontent en premier, les scores bas passent direct en "autres").
- **LLM** → réserve-le à deux usages ciblés plutôt qu'à la classification de masse : (a) générer/suggérer automatiquement les listes de mots-clés par sous-thème à partir de la page Cerema (travail ponctuel, pas dans la boucle de run), (b) traiter les cas ambigus résiduels après méthodes 2+3, en petit volume, avec justification textuelle demandée (ça redonne un peu d'explicabilité).

Le comité d'experts, c'est votre file Grist de validation : au fil des sessions, vous alimentez à la fois le jeu de vérité *et* vous enrichissez les listes de mots-clés (faux négatifs → mots manquants ; faux positifs → mots trop génériques à retirer).

## Modèle de configuration (extensible comme demandé)

Fichiers YAML versionnés dans le repo (pas en dur dans le code) :

```yaml
# themes.yaml
- id: trafic_routier
  label: "Trafic routier"
  keywords: [trafic, circulation, comptage routier, tmja, capteur trafic]
  sous_themes:
    - id: capteurs
      label: "Capteurs de comptage"
      keywords: [capteur, boucle magnétique, spire]
    - id: donnees_mesurees
      label: "Données mesurées de trafic"
      keywords: [débit, vitesse moyenne, tmja]
    - id: autres
      label: "Autres"
```

Ça répond directement à tes 3 besoins d'extensibilité (thèmes, sous-thèmes, mots-clés) sans toucher au code.

Je te propose un schéma du pipeline complet avec la boucle de validation, ça va clarifier où chaque brique s'insère.## Plan d'itération concret, en 4 phases

**Phase 1 — pipeline squelette fonctionnel (1-2 semaines)**
Extraction data.gouv large sur mots-clés génériques mobilité (rappel élevé, tant pis pour le bruit), classification thème/sous-thème par simple recherche de mots-clés dans titre/description/tags, publication brute dans Grist avec un score et un statut `candidat_auto`. Objectif : avoir quelque chose en Grist à regarder, même moyen.

**Phase 2 — curation manuelle intensive**
Vous passez en revue par thème dans Grist (filtres, tri par score). Chaque décision (valide/rejette/ajout manuel) s'écrit dans la table de vérité. Ce travail constitue en même temps votre **jeu d'évaluation** pour la suite.

**Phase 3 — amélioration outillée**
Avec ce jeu de vérité, vous pouvez : mesurer précision/rappel de la méthode mots-clés, enrichir les listes de mots-clés là où il y a des trous, et **évaluer objectivement** l'apport des embeddings (est-ce que le score sentence-transformers sépare mieux les vrais positifs des faux positifs que les mots-clés seuls ?). Le LLM peut être testé ici spécifiquement sur les cas où mots-clés et embeddings sont en désaccord — c'est un sous-ensemble petit et gérable, adapté à une API gratuite ou un modèle local (ex. via Ollama) si pas de licence payante.

**Phase 4 — industrialisation**
Une fois la combinaison de méthodes stabilisée sur le thème pilote (trafic routier), vous répliquez sur les autres thèmes, avec le même mécanisme de fusion table-de-vérité pour ne jamais perdre le travail déjà validé.

Sur la notification des producteurs, je recommande de ne pas l'automatiser complètement tout de suite : générer une liste de suggestions (mots-clés manquants, dataset mal classé côté producteur) et la faire valider manuellement avant tout envoi, au moins le temps que la classification soit fiable — un email basé sur un faux positif serait contre-productif pour la crédibilité de la démarche.

Veux-tu que je te prépare une ébauche de code Python pour le squelette de la Phase 1 (extraction data.gouv + classification mots-clés + structure de config YAML + écriture vers Grist) ?
