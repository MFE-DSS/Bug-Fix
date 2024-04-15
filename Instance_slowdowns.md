# Migration vers PostgreSQL pour les Bases de Données de Runtime dans Dataiku DSS

## Contexte

Vous observez des problèmes de ralentissement au niveau de l'application ont été observés. Bien que le serveur (de l'instance) et les bases de données sous-jacente ne présentent aucun problème, vous constatez des difficultés lors de l'exploration des datasets : l'échantillonnage ne se termine pas et aboutit à une erreur après quelques minutes.
Une solution palliative proposée est la migration de la base de données de runtime du moteur H2 vers PostgreSQL.

## Configuration de l'Internal Database PostgreSQL

La migration vers une base de données PostgreSQL nécessite une attention particulière aux paramètres de configuration dans le fichier `config/general-settings.json` de Dataiku DSS. Exemple de paramètres clés à configurer pour la base de données PostgreSQL :

```json
"internalDatabase": {
    "connection": {
        "params": {
            "port": 15432,
            "host": "HOST_OF_YOUR_POSTGRESQL_DATABASE",
            "user": "USER_OF_YOUR_POSTGRESQL_DATABASE",
            "password": "PASSWORD_OF_YOUR_POSTGRESQL_DATABASE",
            "db": "DB_OF_YOUR_POSTGRESQL_DATABASE"
        },
        "type": "PostgreSQL"
    },
    "tableNamePrefix": "optional prefix to prepend to all table names",
    "schema": "Name of the PostgreSQL schema in which DSS will create its tables",
    "externalConnectionsMaxIdleTimeMS": 600000,
    "externalConnectionsValidationIntervalMS": 180000,
    "maxPooledExternalConnections": 50,
    "builtinConnectionsMaxIdleTimeMS": 1800000,
    "builtinConnectionsValidationIntervalMS": 600000,
    "maxPooledBuiltinConnectionsPerDatabase": 50
}
```
**## Ajustements Recommandés**

Pour améliorer la résilience aux déconnexions, il est conseillé de réduire l'intervalle de validation de 180000 à 30000 et le temps maximal inactif à 60000. Ces ajustements permettent de mieux gérer les connexions interrompues, même si celles-ci ne sont pas toujours consignées dans les journaux par défaut.
Clés de Configuration Spécifiques

    externalConnectionsMaxIdleTimeMS : Temps maximal (en millisecondes) pendant lequel une connexion externe peut rester inactive avant d'être fermée. Valeur recommandée : 300000.
    externalConnectionsValidationIntervalMS : Intervalle (en millisecondes) au bout duquel une connexion externe est validée. Valeur recommandée : 30000.

## Migration vers PostgreSQL et Maintenance de Dataiku DSS

### Copier les Bases de Données vers un Emplacement Externe

Pour déplacer les bases de données internes vers un stockage externe, utilisez :

```bash
./bin/dssadmin copy-databases-to-external
./bin/dssadmin encrypt-password YOUR-PASSWORD
```
exemple chiffrement aes 
```bash
./bin/dssadmin encrypt-password e:AES:instance-name#interne#25
```
## Bonus : script de nettoyage des fichiers temporaires

Au délà de la database du runtime maintenant deportée dans postgrel il est possible que les fichiers temporaires et streamings logs viennent causer des ralentissements du même ordre.

### Mise en Place d'un Script de Nettoyage

Le script suivant purge les métriques du moteur DSS  > 30 jours, situées dans /tmp/ :

Contenu du script /data/dataiku/scripts/nettoyage/mon_fichier_de_nettoyage-engine-metrics.sh : 
#!/bin/bash
## script de purge des dss engine metrics à 30 jours

find /data/dataiku/design/tmp/dss-engine-metrics -mtime +30 -exec rm {} \;
find /data/dataiku/design/tmp/dss-engine-metrics -mtime +30 -exec rmdir {} \;

## Pour exécuter ce script automatiquement tous les jours, ajoutez-le à la crontab de l'utilisateur Dataiku :

0 4 * * * /data/dataiku/scripts/nettoyage/mon_fichier_de_nettoyage-engine-metrics.sh

Cette ligne exécute le script de purge tous les jours à 4h00 du matin.
