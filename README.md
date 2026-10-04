# Steam : analyse du marché du jeu vidéo pour Ubisoft

## Le projet

Ubisoft prépare un nouveau jeu et veut comprendre le marché avant de le lancer. Nous
avons analysé le catalogue de la plateforme Steam, qui compte 55 690 jeux, pour décrire
ce marché à trois niveaux : l'ensemble du catalogue, les genres, les plateformes.

C'est une analyse exploratoire : nous décrivons et nous comparons, nous ne prédisons
rien. Elle est réalisée avec PySpark (la version Python de Spark, un outil de calcul
distribué) sur Databricks, dont nous utilisons aussi l'outil de visualisation.

Le livrable est un seul notebook, steam.ipynb, en sept chapitres. Projet réalisé dans le
cadre de la certification Concepteur Développeur en Science des Données, bloc 2.

## Les données

- **Un fichier JSON unique**, fourni par Jedha, lu directement depuis son emplacement
  S3 dans le notebook. Une copie de référence est conservée dans data/raw/.
- **Une ligne par jeu**, 55 691 au total, avec 22 informations par jeu : nom, éditeur,
  développeur, genres, langues, prix, remise, date de parution, plateformes, âge requis,
  avis positifs et négatifs, fourchette de possesseurs, pic de joueurs connectés.
- **Les noms de colonnes sont ceux de l'API SteamSpy**, un site de statistiques sur
  Steam. Les ventes ne sont pas dans les données : la popularité est approchée par le
  nombre de possesseurs et par les avis.

## Comment ça marche

1. **Chargement et inspection** : lecture du fichier en texte brut pour en comprendre
   la forme, puis en JSON ; Spark déduit la structure et crée une ligne par jeu.
2. **Mise à plat et nettoyage** : les informations de chaque jeu, rangées dans un bloc,
   deviennent des colonnes. Prix, remise, date de parution, fourchette de possesseurs
   et âge requis sont écrits en texte dans le fichier : nous les convertissons en nombres
   et en dates, après avoir listé toutes les formes qu'ils prennent.
3. **Tables par niveau d'analyse** : un jeu a plusieurs genres, plusieurs langues,
   plusieurs plateformes dans une seule case. Nous construisons une table par niveau,
   avec une ligne par couple jeu et genre, jeu et langue, jeu et plateforme.
4. **Analyse d'ensemble** : éditeurs, jeux les mieux notés, parutions par année, prix
   et remises, langues, limites d'âge.
5. **Analyse des genres** : genres les plus représentés, part d'avis positifs par genre.
6. **Analyse des plateformes** : disponibilité sur Mac et Linux par genre.
7. **Synthèse pour Ubisoft**.

Trois choix ont compté :

- **Rien n'est inventé, rien n'est supprimé.** Les 222 jeux sans date complète et les
  5 jeux à l'âge requis impossible (180 ans, 35 ans) restent dans la table, avec la
  case vide. Nous gardons ce que les données disent et laissons vide ce qu'elles ne
  disent pas.
- **Le seuil d'avis est calculé, pas choisi.** Un jeu avec trois avis positifs serait à
  100 %. Nous ne classons que les jeux ayant plus d'avis que le jeu le plus commenté
  sans aucun avis négatif, soit 496 avis : au-dessus, chaque jeu a reçu au moins une
  critique.
- **Les genres de logiciels sont écartés.** Steam vend aussi des logiciels, rangés comme
  des jeux. Aucune catégorie ne compte entre 700 et 1 400 titres, et tout ce qui est en
  dessous est un logiciel ou un avertissement de contenu : nous gardons les douze genres
  d'au moins 1 000 jeux.

## Les résultats

| Question | Réponse |
|---|---|
| Éditeur le plus prolifique | Big Fish Games, 422 jeux ; Ubisoft neuvième avec 127 |
| Structure du marché | Près de 30 000 éditeurs, les dix premiers pèsent 3 % du catalogue |
| Parutions par année | 96 % des jeux parus depuis 2014 ; records en 2020 et 2021 (plus de 8 000 par an) |
| Prix | Médiane 4,99 $, 14 % de jeux gratuits, 1 % au-dessus de 40 $ |
| Remises | 4,5 % des jeux en promotion, remise médiane de 60 % |
| Langues | Anglais dans 99 % des jeux, français dans 24 % |
| Limites d'âge | 1,2 % des jeux en déclarent une, 0,4 % interdits aux moins de 18 ans |
| Genres | Indie dans 71 % des jeux ; part d'avis positifs de 80 à 89 % selon le genre, sauf Massively Multiplayer à 73 % |
| Plateformes | Windows pour 99,97 % des jeux, Mac 23 %, Linux 15 % ; stratégie et indépendants sont les plus portés sur Mac |

Les jeux les mieux notés dépassent tous 99 % d'avis positifs et aucun n'est une grosse
production. Le marché est fragmenté, récent, et bon marché.

## Exécuter le projet

Arborescence du dépôt :

- README.md
- steam.ipynb, le notebook
- data/raw/steam_game_output.json, la copie de référence des données

Prérequis : un compte Databricks (l'édition gratuite suffit). Aucune configuration ni
clé n'est nécessaire : le fichier est lu depuis une adresse publique.

Lancement :

1. Importer steam.ipynb dans l'espace de travail Databricks (File > Import).
2. Rattacher le notebook à un calcul et lancer Run all.
3. Après une pause, le calcul s'arrête et les variables sont perdues : relancer
   les cellules du chapitre 1 puis celles qui construisent les tables.

Les graphiques sont créés avec l'outil de visualisation de Databricks. Ils s'affichent
dans Databricks mais ne sont pas conservés dans le fichier exporté : les tableaux qui
les alimentent y restent.

## Limites et suites possibles

- **Les ventes ne sont pas dans les données** : nous les approchons par la fourchette de
  possesseurs et le nombre d'avis, deux mesures imparfaites.
- **Les données s'arrêtent au 11 novembre 2022** : l'année 2022 est incomplète et le
  marché a pu changer depuis.
- **Genres et limites d'âge sont déclarés par les éditeurs**, pas vérifiés : les limites
  d'âge sont probablement sous-estimées.
- **Une vingtaine de langues mal saisies** sont laissées telles quelles : elles ne pèsent
  rien sur le classement.
- **Pistes non traitées** : les genres favoris de chaque éditeur, une estimation de ce
  que rapporte chaque genre, et les tags des joueurs, laissés de côté.
