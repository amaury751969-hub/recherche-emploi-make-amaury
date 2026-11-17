# Automatiser ma recherche d’emploi Data Analyst

**Auteur :** Amaury Ducoulombier  
**Date :** 17 novembre 2026  
**Outils :** Make, API JSearch (OpenWeb Ninja), Google Sheets et Discord

## 1. Objectif du projet

Ce projet automatise la collecte d’offres de Data Analyst afin de gagner du temps et de centraliser le suivi de mes candidatures. Le scénario recherche les offres, les ajoute dans Google Sheets et envoie une alerte par offre dans un salon Discord privé.

## 2. Outils utilisés

| Outil | Rôle |
|---|---|
| Make | Relier les services et exécuter le scénario automatiquement |
| API JSearch — OpenWeb Ninja | Fournir les offres correspondant aux critères de recherche |
| Google Sheets | Enregistrer les offres et suivre mes candidatures |
| Discord | Recevoir une alerte avec les informations et le lien pour postuler |

## 3. Critères de recherche

| Paramètre | Valeur configurée |
|---|---|
| query | data analyst Paris |
| country | fr |
| language | fr |
| date_posted | week : offres des sept derniers jours |
| num_pages | 1 |

La recherche porte sur les postes de Data Analyst à Paris. Aucun type de contrat particulier n’est imposé dans les paramètres présentés : les résultats peuvent notamment comprendre des stages et des emplois à plein temps.

![Paramètres JSearch : métier, ville, pays et langue](Images/01-parametres-jsearch.png..png)

*Figure 1 — Paramètres visibles : recherche « data analyst Paris », pays « fr » et langue « fr ».*

![Période et nombre de pages de la recherche JSearch](Images/02-parametres-periode.png..png)

*Figure 2 — Offres des sept derniers jours (date_posted = week), sur une page de résultats (num_pages = 1).*

## 4. Fonctionnement du scénario

Le scénario suit cette chaîne :

**HTTP JSearch → Iterator → Google Sheets → HTTP Discord**

### 4.1. Module HTTP — Rechercher les offres

Le premier module envoie une requête GET à l’adresse `https://api.openwebninja.com/jsearch/search-v2`. Les critères sont transmis dans les paramètres de recherche. La clé d’API est envoyée dans le header `x-api-key` pour identifier le compte et autoriser l’appel.

L’option Parse response permet à Make de transformer la réponse JSON en champs utilisables. Le test a renvoyé un code HTTP 200 et une liste `jobs` contenant dix offres.

### 4.2. Iterator — Parcourir les offres

L’Iterator reçoit la liste `jobs[]` du premier module. Il transmet chaque offre séparément aux modules suivants. Un bundle correspond ici aux données d’une offre.

### 4.3. Google Sheets — Enregistrer les offres

Le module Add a Row ajoute une ligne par offre dans le tableau de suivi. Le mapping associe chaque champ reçu à la colonne correspondante.

| Colonne | Donnée utilisée |
|---|---|
| Date de publication | job_posted_at_datetime_utc, formatée en JJ/MM/AAAA |
| Intitulé | job_title |
| Entreprise | employer_name, ou « Non communiqué » |
| Lieu | job_location, ou « Non communiqué » |
| Type de contrat | job_employment_type |
| Site d’origine | job_publisher |
| Lien pour postuler | job_apply_link |

Les colonnes Statut, Date de candidature et Notes sont placées à droite. Make les laisse vides : elles sont réservées à mon suivi personnel.

### 4.4. HTTP Discord — Envoyer les alertes

Le dernier module envoie une requête POST à un webhook du salon Discord. Le corps est de type `multipart/form-data` et contient un champ texte nommé `content`.

Le message présente l’intitulé du poste, l’entreprise, le lieu, le type de contrat et le lien pour postuler. Une alerte est envoyée pour chaque offre ajoutée.

![Scénario Make complet : HTTP JSearch, Iterator, Google Sheets et HTTP Discord](Images/03-scenario-complet.png.png)

*Figure 3 — Les quatre modules du scénario, de la recherche JSearch au message Discord. La partie confidentielle du chemin du webhook est masquée.*

## 5. Nettoyage et préparation des données

La fonction `formatDate` transforme les dates de publication au format JJ/MM/AAAA. Dans Google Sheets, un format de date personnalisé JJ/MM/AAAA a été appliqué à la colonne pour afficher correctement les dates déjà enregistrées.

La fonction `ifempty` remplace une entreprise ou un lieu absent par « Non communiqué ». Cela évite les cellules vides et rend les informations manquantes explicites.

Le site d’origine permet d’identifier la plateforme qui publie l’offre. Le lien pour postuler est enregistré sous forme cliquable.

## 6. Tests et résultats

| Vérification | Résultat observé |
|---|---|
| Appel de l’API | Code HTTP 200 |
| Liste des offres | Dix offres reçues lors du test |
| Stockage | Dix lignes ajoutées dans Google Sheets |
| Dates | Affichage en JJ/MM/AAAA |
| Suivi personnel | Colonnes Statut, Date de candidature et Notes laissées vides |
| Discord | Envoi des alertes réussi |
| Exécution complète | 22 crédits Make, durée de 11 secondes |

Les lignes de test ont ensuite été supprimées du tableau en conservant les en-têtes.

![Tableau de suivi rempli : dix offres et colonnes de suivi personnel](Images/04-tableau-offres.png.png)

*Figure 4 — Dix offres enregistrées dans Google Sheets, avec les dates au format JJ/MM/AAAA et les colonnes de suivi personnel laissées vides.*

![Alertes reçues dans le salon Discord offres_data](Images/05-alertes-discord.png..png)

*Figure 5 — Alertes reçues dans le salon offres_data : intitulé, entreprise, lieu, contrat et lien pour postuler.*

## 7. Planification et consommation

### 7.1. Planification

Le scénario est configuré pour une exécution chaque lundi à 08:00, dans le fuseau Europe/Paris. Son état Active est visible sur la capture.

![Planification hebdomadaire du scénario Make](Images/06-planification.png..png)

*Figure 6 — Exécution chaque lundi à 08:00, fuseau Europe/Paris ; scénario actif.*

### 7.2. Crédits Make

L’exécution complète du 10 novembre 2026 à 12:18:36 a consommé 22 crédits pour dix offres.

| Module | Crédits |
|---|---:|
| HTTP — recherche JSearch | 1 |
| Iterator | 1 |
| Google Sheets — dix ajouts | 10 |
| HTTP Discord — dix alertes | 10 |
| **Total** | **22** |

Avec cinq exécutions mensuelles et environ dix offres par exécution, la consommation estimée est de **22 × 5 = 110 crédits Make par mois**. Elle varie avec le nombre d’offres : pour N offres, cette version consomme 2 + 2N crédits par exécution.

### 7.3. Requêtes JSearch

Avec une recherche sur une page, l’estimation est d’une requête JSearch par exécution, soit **cinq requêtes par mois** pour cinq lundis. Le compteur observé après les essais indiquait trois requêtes consommées sur les 200 de l’offre Basic décrite dans le brief.

Ces estimations mensuelles excluent les tests et la consommation du scénario de veille RSS. Il faut conserver une marge dans le quota Make commun aux scénarios.

## 8. Difficultés rencontrées et solutions

| Difficulté | Solution |
|---|---|
| Le tableau n’apparaissait pas dans le sélecteur de fichiers | Vérification du compte Google et de l’emplacement du fichier ; sélection du tableau accessible dans My Drive |
| Le champ Sheet Name était vide | Sélection de l’onglet « Feuille 1 » |
| Les dates reçues n’étaient pas au format demandé | Application de formatDate avec le format DD/MM/YYYY |
| Risque de cellules vides pour l’entreprise ou le lieu | Application de ifempty avec la valeur « Non communiqué » |
| Composition du message Discord | Saisie du champ content et insertion des pastilles de l’Iterator dans le message |

## 9. Sécurité et limites

La clé d’API et l’URL du webhook sont des secrets. Elles ne doivent apparaître ni dans le document public, ni dans le blueprint partagé, ni sur les captures.

Une copie du blueprint, nommée blueprint-sans-secrets.json, a été préparée : la clé API et le webhook ont été remplacés par ***. Le fichier JSON et la présence des quatre modules ont été vérifiés. Les captures intégrées ne montrent pas ces secrets.

La recherche est limitée à une page de résultats. Une exécution hebdomadaire sur les sept derniers jours réduit les chevauchements, mais ne garantit pas l’absence de doublons. Un lancement manuel supplémentaire peut réinsérer des offres et renvoyer les alertes.

Le bonus de détection des doublons n’a pas été intégré à la version présentée. Il pourrait rechercher une offre déjà enregistrée avant de poursuivre l’ajout et l’envoi Discord.

## 10. Liens et livrables

**Google Sheet en lecture seule :** [Consulter le tableau de suivi des candidatures](https://docs.google.com/spreadsheets/d/1nULDE8K676zLpUiuuRBXo8bRHllV9BB3NniWqwg_CvY/edit?usp=sharing)

**Dépôt GitHub public :** [recherche-emploi-make-amaury](https://github.com/amaury751969-hub/recherche-emploi-make-amaury)

Le dépôt doit contenir directement, sans archive ZIP :

- le document de présentation au format Word (.docx) ou texte (.txt) ;
- le blueprint JSON sans clé d’API ni URL de webhook ;
- les captures du scénario complet, du tableau rempli, des alertes Discord et de la planification.
