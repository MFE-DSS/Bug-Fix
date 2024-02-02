# Correction de Bug : Traitement de Datasets ORC avec Spark dans Dataiku

## Contexte du Problème

Lors de l'utilisation de Dataiku DSS (Dataiku FE) pour synchroniser des tables partitionnées au format ORC (construites avec Hive) en utilisant l'engine Spark, les utilisateurs peuvent rencontrer un problème où le dataset de sortie est vide. Ce problème ne se présente pas lors de l'utilisation de l'engine Hive, ni lors de l'utilisation de Spark avec des datasets partitionnés au format CSV.

### Exemple de Problème

- **Incident / Défaut** : Synchronisation/Build d'un dataset HDFS partitionné en format ORC avec l'engine Spark produit un dataset vide.

## Description du Problème

Le lecteur ORC natif dans Spark ne supporte pas les sous-répertoires pour les tables Hive, ce qui peut causer des problèmes de lecture des données ORC partitionnées lors de l'exécution avec Spark.

## Solution Trouvée

La solution à ce problème est documentée dans le JIRA Spark sous l'identifiant [SPARK-28098](https://issues.apache.org/jira/browse/SPARK-28098), indiquant que le lecteur ORC natif ne supporte pas les sous-répertoires avec les tables Hive.

### Correction à Appliquer

Pour corriger ce problème, il est nécessaire de désactiver la conversion par défaut des tables ORC par Hive en modifiant la configuration de Spark. Cela peut être fait de deux manières dans Dataiku :

#### Dans les Clés de Configuration des Engines Spark

Ajouter la clé de configuration suivante :
```yaml
spark.sql.hive.convertMetastoreOrc: false

Dans la Configuration Avancée des Datasets

Dans les paramètres avancés des datasets Dataiku, ajouter la configuration Spark suivante :

set spark.sql.hive.convertMetastoreOrc=False

ou 

spark.sql.hive.convertMetastoreOrc -> false

