# Depreciation de la fonction get_definition() et necessité de refactoring de scripts avec get_settings()
https://developer.dataiku.com/12/api-reference/python/users-groups.html#dataikuapi.dss.admin.DSSUser.get_definition

Rappel.\
-> Pour itérer sur les éléments d'un dictionnaire en Python, vous utilisez généralement une boucle for. \
-> Les dictionnaires en Python sont des structures de données qui stockent des paires clé-valeur. \
-> Lorsque vous itérez sur un dictionnaire, vous pouvez choisir d'itérer sur ses clés, ses valeurs, ou sur les deux en même temps. \

```python
Itération sur les clés d'un dictionnaire:
for key in mon_dictionnaire:
    print(key)  # Affiche la clé
#Ou:

for key in mon_dictionnaire.keys():
    print(key)  # Affiche la clé
Itération sur les valeurs d'un dictionnaire:
for value in mon_dictionnaire.values():
    print(value)  # Affiche la valeur
Itération sur les paires clé-valeur d'un dictionnaire:
for key, value in mon_dictionnaire.items():
    print(key, value)  # Affiche la clé suivie de la valeur
```
### Application avec l'API Python de Dataiku
Supposons que vous récupériez des informations d'utilisateur ou configurations de façon brute avec get_settings().get_raw(), qui renvoie un dictionnaire:
```python
user_settings = user.get_settings().get_raw()
Vous pouvez itérer sur les configurations de l'utilisateur comme ceci:

for setting_key, setting_value in user_settings.items():
    print(f"La configuration {setting_key} a la valeur {setting_value}")
```
C'est une méthode utile pour, par exemple, débuguer ou logger l'intégralité des configurations, ou encore pour appliquer des transformations ou vérifications spécifiques sur chaque élément de configuration.

## Travailler Avec Des Dictionnaires Imbriqués
Parfois, les dictionnaires contiennent d'autres dictionnaires ou même des listes comme valeurs. L'itération peut alors se complexifier:
```python
for key, value in mon_dictionnaire_complexe.items():
    if isinstance(value, dict):
        # c'est un sous-dictionnaire, itérez sur lui
        for subkey, subvalue in value.items():
            print(key, subkey, subvalue)
    elif isinstance(value, list):
        # c'est une liste, peut-être de dictionnaires
        for item in value:
            if isinstance(item, dict):
                # c'est un dictionnaire, itérez sur lui
                for item_key, item_value in item.items():
                    print(key, item_key, item_value)
            else:
                # c'est une valeur seule
                print(key, item)
```

## L'API Python de Dataiku DSS fournit différentes méthodes pour accéder aux informations associées aux utilisateurs.
Les méthodes get_definition() et get_settings() sont utilisées pour des objectifs similaires mais avec des différences subtiles dans les versions de l'API et dans les types de résultats retournés.


## get_definition()
Cette méthode est souvent utilisée pour obtenir une "définition" d'un objet dans Dataiku, ce qui peut inclure un utilisateur. Elle renvoie généralement un dictionnaire (ou dans certains cas, une liste de dictionnaires) contenant les détails de cet objet.
L'utilisation de get() sur le résultat de get_definition() sera efficace si le résultat est de type dict. S'il s'agit d'une liste, cela implique généralement que l'objet a plusieurs "définitions", et vous devrez itérer sur cette liste ou accéder à un élément spécifique par index avant de pouvoir utiliser get().
## get_settings()
La méthode get_settings() est utilisée pour obtenir les paramètres ou la configuration d'un objet Dataiku. Pour un utilisateur, cela renverrait un objet de configuration contenant les paramètres spécifiques à cet utilisateur.
### get_raw() est une méthode pour obtenir ces paramètres sous la forme d'un dictionnaire brut (non traité). Vous pouvez appeler get_raw() sur le résultat de get_settings() pour accéder à ces valeurs sous forme de dictionnaire. 
Si vous essayez d'appeler get() directement sur le résultat de get_settings(), cela ne fonctionnera pas car get_settings() ne renvoie pas directement un dictionnaire mais un objet contenant le dictionnaire.
Pour le cas où get_definition() renvoie une liste, si vous avez besoin de récupérer les groupes, vous devrez manipuler ces données comme une liste. Voici un exemple pour clarifier :

```python
user = client.get_user(user_login)
user_definition = user.get_definition()
if isinstance(user_definition, list):
    # Supposons que nous voulons les groupes du premier élément de définition
    user_groups = user_definition[0].get('groups', [])
else:
    # La définition est un dictionnaire
    user_groups = user_definition.get('groups', [])
```
Quant à l'apparence dépréciée de get_definition(), si get_settings() lui est précisé comme successeur, vous devriez prioriser l'utilisation de cette dernière méthode pour les nouvelles implémentations. 
Cette méthode vous permet de vous "enfoncer" dans des structures de données imbriquées pour extraire ou manipuler les informations dont vous avez besoin.
