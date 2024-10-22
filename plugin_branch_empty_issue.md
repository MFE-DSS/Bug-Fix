# Guide de Résolution des Problèmes de Plugins Dataiku
## Ce guide fournit une procédure générale pour résoudre les problèmes de plugin dans Dataiku, en particulier lorsque le plugin est associé à une branche Git inappropriée ou vide, ce qui rend le plugin invisible mais non supprimable.

### Étape 1: Accéder à la VM
Connectez-vous à la machine virtuelle Ubuntu 22.04 via SSH.
```bash
ssh your_user@your_vm_ip
```
### Étape 2: Naviguer vers le Répertoire des Plugins
Dirigez-vous vers le répertoire des plugins de Dataiku.
```bash
cd /path/to/dataiku/plugins/dev/
```
Remplacez /path/to/dataiku/plugins/ par le chemin réel où vos plugins Dataiku sont installés.

### Étape 3: Vérifier l'État Actuel du Plugin
Listez les plugins présents et vérifiez l'état du plugin problématique.
```bash
ls -la plugin-name
cd plugin-name
git status
```
### Étape 4: Corriger la Branche Git
Si le plugin est sur une branche incorrecte, commencez par récupérer les dernières informations du référentiel Git.
```bash
git fetch origin
```
Ensuite, changez pour la branche correcte qui contient une version valide du plugin.
```bash
git checkout THE_CORRECT_BRANCH
git pull
```
Remplacez THE_CORRECT_BRANCH par le nom de la branche appropriée.

### Étape 5: Redémarrer Dataiku
Pour que Dataiku prenne en compte les modifications, redémarrez le service.
```bash
sudo systemctl restart dataiku  # Si Dataiku est géré par systemd
# ou
/path/to/dataiku/dss/bin/dss restart  # Si Dataiku est démarré via un script customz
```
### Étape 6: Supprimer le Plugin si Nécessaire
Si le plugin doit être supprimé pour une réinstallation propre, faites-le avec prudence (après avoir fait une sauvegarde).
```bash
rm -rf /path/to/dataiku/plugins/dev/plugin-name
```

### Recommandations Générales

#### - Évitez les changements vers des branches vides : Ne basculez jamais sur une branche Git qui ne contient pas de version valide du plugin.
#### - Utilisez des branches stables : Assurez-vous que chaque branche utilisée contient une version complète et fonctionnelle du plugin.
#### - Maintenez des sauvegardes régulières : Avant toute manipulation significative, vérifiez que vous avez une sauvegarde des plugins et de leurs configurations.
#### - Contrôle des versions : Supervisez et documentez rigoureusement les versions des plugins et les branches Git associées pour éviter les conflits et les erreurs.
