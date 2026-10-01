# Account Locator — Salesforce DX

Application Salesforce de recherche et de localisation de comptes sur une carte, réalisée dans le cadre du module Trailhead [Développement d’applications avec Salesforce DX](https://trailhead.salesforce.com/fr/content/learn/modules/sfdx_app_dev).

Ce dépôt présente mon travail d’apprentissage : mise en place du projet local, réalisation guidée des composants, déploiement dans des scratch orgs et manipulation des données avec Salesforce CLI. L’application et son code de base suivent le tutoriel Salesforce ; il ne s’agit pas d’une application conçue intégralement de zéro.

## Objectif

Comprendre le cycle de développement Salesforce DX en construisant une application qui relie une interface Aura à une recherche Apex, puis en reproduisant l’application dans une nouvelle organisation de test à partir des fichiers du projet.

## Fonctionnalités

- Rechercher des comptes avec une requête SOSL exécutée côté Apex.
- Afficher jusqu’à 10 résultats, triés par nom, dans un tableau.
- Consulter le téléphone et le site web des comptes trouvés.
- Ouvrir la fiche d’un compte depuis le bouton **View details**.
- Afficher les résultats sur une carte à partir de leur adresse de facturation.
- Accéder à l’application via l’onglet **Account Locator** et l’ensemble d’autorisations **Geolocation**.

La recherche initiale utilise `San Francisco`. Les trois comptes d’exemple sont **Marriott Marquis**, **Hilton Union Square** et **Hyatt**, fournis dans `data/Account.json`.

## Technologies et notions pratiquées

| Technologie ou notion | Utilisation dans ce projet |
| --- | --- |
| Apex et SOSL | Recherche des comptes dans `AccountSearchController` |
| Aura et JavaScript | Interface, appels au serveur et traitement des résultats |
| Événement Aura | Transmission des résultats avec `AccountsLoaded` |
| `lightning:datatable` | Tableau et action d’ouverture d’une fiche |
| `lightning:map` | Affichage des marqueurs à partir des adresses |
| Salesforce CLI | Création d’orgs, déploiement, récupération des métadonnées et import/export des données |
| Dev Hub et scratch orgs | Gestion d’environnements temporaires de développement et de validation |
| Permission Set | Visibilité de l’onglet Account Locator |

Le tutoriel utilise **Aura**. Ce projet ne contient pas d’interface Lightning Web Components (LWC).

## Fonctionnement

1. `AccountLocator` rassemble la recherche, la liste et la carte.
2. `AccountSearch` appelle la méthode Apex `searchAccounts` avec le texte saisi.
3. `AccountSearchController` renvoie les comptes trouvés par SOSL. Une recherche vide renvoie une liste vide.
4. Le composant publie l’événement `AccountsLoaded` avec les résultats.
5. `AccountList` met à jour le tableau et `AccountMap` construit les marqueurs de la carte.

Les marqueurs utilisent `BillingStreet`, `BillingCity` et `BillingPostalCode`. Il n’est pas nécessaire de saisir des coordonnées GPS pour les données d’exemple.

## Structure du dépôt

```text
config/
  project-scratch-def.json       # Configuration d’une scratch org
data/
  Account.json                  # Trois comptes d’exemple du tutoriel
force-app/main/default/
  classes/
    AccountSearchController.cls # Recherche Apex / SOSL
  aura/
    AccountLocator/             # Composant principal
    AccountSearch/              # Saisie et lancement de la recherche
    AccountsLoaded/             # Événement de résultats
    AccountList/                # Tableau et navigation
    AccountMap/                 # Carte et marqueurs
  permissionsets/
    Geolocation.permissionset-meta.xml
  tabs/
    Account_Locator.tab-meta.xml
sfdx-project.json               # Configuration du projet DX
```

## Reproduire le projet

### Prérequis

- Git et Salesforce CLI (`sf`).
- Une organisation Salesforce avec **Dev Hub activé** et une allocation de scratch orgs disponible.
- VS Code avec Salesforce Extension Pack pour travailler sur le code.
- Une version Salesforce compatible avec la version d’API déclarée dans `sfdx-project.json` (actuellement `67.0`).

Cloner ce dépôt avec l’URL fournie par le bouton **Code** de GitHub, puis ouvrir son dossier dans le terminal. Toutes les commandes suivantes s’exécutent à la racine du projet, où se trouve `sfdx-project.json`.

### 1. Connecter le Dev Hub

```powershell
sf org login web --set-default-dev-hub --alias DevHub
```

Se connecter dans le navigateur avec les identifiants de l’org où Dev Hub est activé.

### 2. Créer une scratch org

```powershell
sf org create scratch --definition-file config/project-scratch-def.json --alias GeoPortfolio --target-dev-hub DevHub --set-default --duration-days 7
```

### 3. Déployer les métadonnées

```powershell
sf project deploy start --target-org GeoPortfolio
```

### 4. Attribuer l’ensemble d’autorisations

```powershell
sf org assign permset --name Geolocation --target-org GeoPortfolio
```

### 5. Importer les données d’exemple

```powershell
sf data import tree --files data/Account.json --target-org GeoPortfolio
```

Effectuer cet import une seule fois dans cette org pour éviter les doublons. Les métadonnées et les données sont transférées séparément : un déploiement du code ne crée pas les trois comptes.

### 6. Ouvrir l’application

```powershell
sf org open --target-org GeoPortfolio
```

Dans le lanceur d’applications, rechercher **Account Locator**. Essayer `San Francisco`, puis `Marriott`, et vérifier la liste, la carte et le bouton **View details**. L’indexation SOSL peut demander un délai après l’import.

Les scratch orgs sont temporaires : cette application n’a pas de démonstration publique permanente. Les commandes ci-dessus permettent de recréer un environnement.

## Validation et limites actuelles

L’affichage des comptes sur la carte a été vérifié manuellement dans `GeoTestOrg` lors de la réalisation du module.

Le dépôt ne contient pas encore de tests Apex automatisés ni de pipeline CI/CD. Les fichiers de configuration Jest provenant du modèle Salesforce DX ne constituent pas une suite de tests pour cette application Aura. Ce projet est destiné à l’apprentissage en scratch org, pas à un déploiement direct en production.

Le contrôleur Apex utilise `with sharing`. Une revue complémentaire des permissions objet/champ et de l’accès à la classe Apex serait nécessaire pour une utilisation avec des profils métier : l’ensemble d’autorisations actuel configure uniquement la visibilité de l’onglet.

## Difficultés rencontrées et apprentissages

- **Terminal Windows :** une commande de création de compte échouait dans Git Bash avec une erreur sur `C:\Program`. Son exécution dans PowerShell a permis de poursuivre l’exercice.
- **Choix de l’organisation :** l’application semblait vide dans un onglet du navigateur. L’ouverture explicite de `GeoTestOrg` avec `--target-org` a permis de retrouver les données attendues.
- **Métadonnées et données :** reproduire l’application nécessite de déployer les composants et d’importer séparément les comptes d’exemple.
- **Synchronisation :** les éléments créés dans l’interface Salesforce, notamment l’onglet et le Permission Set, doivent être récupérés dans le projet pour pouvoir les redéployer.

## Améliorations envisagées

- Ajouter des tests Apex, notamment avec `Test.setFixedSearchResults` pour contrôler les résultats SOSL.
- Gérer explicitement les erreurs des appels Apex et les états « aucun résultat » dans l’interface.
- Temporiser la recherche pendant la saisie afin de limiter les appels serveur.
- Compléter les contrôles de permissions pour des utilisateurs non administrateurs.
- Refaire l’interface en LWC pour comparer les deux modèles de composants.

## Ressources et attribution

- [Module Trailhead — Développement d’applications avec Salesforce DX](https://trailhead.salesforce.com/fr/content/learn/modules/sfdx_app_dev)
- [Création de l’affichage de la carte et finalisation de l’application](https://trailhead.salesforce.com/fr/content/learn/modules/sfdx_app_dev/sfdx_app_dev_create_visuals)
- [Documentation Salesforce DX](https://developer.salesforce.com/docs/platform/sfdx-dev/guide/sfdx-dev-intro.html)

Les exemples de code et les données d’apprentissage sont issus du tutoriel Salesforce. Ce dépôt les rassemble avec une documentation du fonctionnement, de la mise en place et des apprentissages réalisés.
