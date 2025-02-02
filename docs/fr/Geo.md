# Environnement d'analyse géospatial (EGA) - Accès multiplateforme
	
## Mise en route

**IMPORTANT: Prérequis**

	1. Un projet intégré avec accès à  DAS GAE ArcGIS Portal 	
	2. Un id client ArcGIS Portal (clé API)

ArcGIS Enterprise Portal est accessible dans AAW ou CAE à l'aide de l'API, à partir de n'importe quel service qui exploite le langage de programmation Python. 

Par exemple, dans AAW et l'utilisation de [Jupyter Notebooks](https://statcan.github.io/aaw/fr/1-Experiments/Jupyter/) dans l'espace, ou dans CAE l'utilisation de [Databricks](https://statcan.github.io/cae-eac/fr/DataBricks/), DataFactory, etc.

[Le portail Das GAE ArcGIS Enterprise est accessible directement ici](https://geoanalytics.cloud.statcan.ca/portal)

[Pour obtenir de l'aide sur l'auto-inscription en tant qu'utilisateur du portail géospatial DAS]
(https://statcan.github.io/gae-eag/french/portalFR/)

<hr>

## Utilisation d'ArcGIS API (interface de programmation d'application) for Python

### Connexion à ArcGIS Enterprise Portal à l'aide de l'API ArcGIS

1. Installer les packages :

	```python
	conda install -c esri arcgis
	```

	ou en utilisant Artifactory

	```python3333
	conda install -c https://jfrog.aaw.cloud.statcan.ca/artifactory/api/conda/esri-remote arcgis
	```

2. Importez les bibliothèques nécessaires dont vous aurez besoin dans le Bloc-notes.
	```python
	from arcgis.gis import GIS
	from arcgis.gis import Item
	```
	
3. Accéder au portail
	Votre groupe de projet recevra un ID client lors de l'intégration. Collez l'ID client entre les guillemets ```client_id='######'```. 
	
	```python
	gis = GIS("https://geoanalytics.cloud.statcan.ca/portal", client_id=' ')
	print("succès: " + gis.properties.user.username)
	```

4. - Le résultats vous redirigera vers un portail de connexion.
	- Utilisez l'option connexion Azure de StatCan et votre ID cloud 
	- Après une connexion réussie, vous recevrez un code pour vous connecter à l'aide de SAML. 
	- Collez ce code dans le résultat (en anglais seulement). 

	![OAuth2 Approval](https://github.com/StatCan/gae-eag/blob/ghpages/english/images/OAuth2Key.png) (en anglais seulement)

<hr>

### Afficher les informations de l'utilisateur
En utilisant la fonction 'me', nous pouvons afficher diverses informations sur l'utilisateur connecté.
```python
me = gis.users.me
username = me.username
description = me.description
display(me)
```

<hr>

### Recherche de contenu
Recherchez le contenu que vous avez hébergé sur le portail géographique SAD. En utilisant la fonction Â«Â meÂ Â», nous pouvons rechercher tout le contenu hébergé sur le compte. Il existe plusieurs façons de rechercher du contenu. Deux méthodes différentes sont décrites ci-dessous.

**Rechercher tous vos itmes hébergés dans le portail géographique SAD.**
```python
my_content = me.items()
my_content
```
**Recherchez le contenu spécifique que vous possédez dans le portail géographique SAD.**

Ceci est similaire Ã  l'exemple ci-dessus, mais si vous connaissez le titre de la couche qu'ils souhaitez utiliser, vous pouvez l'enregistrer en tant que fonction.
```python
my_items = me.items()
for items in my_items:
    print(items.title, " | ", items.type)
    if items.title == "Flood in Sorel-Tracy":
        flood_item = items
        
    else:
        continue
print(flood_item)
```

**Recherchez tout le contenu auquel vous avez accès, pas seulement le vôtre.**

```python
flood_item = gis.content.search("tags: flood", item_type ="Feature Service")
flood_item
```

<hr>

### Obtenir du contenu
Nous devons obtenir l'élément du portail géographique SAD afin de l'utiliser dans le bloc-notes Jupyter. Pour ce faire, vous fournissez le numéro d'identification unique de l'élément que vous souhaitez utiliser. Trois exemples sont décrits ci-dessous, tous accédant Ã  la couche identique.
```python
item1 = gis.content.get(my_content[5].id) #À partir de la recherche de votre contenu ci-dessus
display(item1)

item2 = gis.content.get(flood_item.id) #De l'exemple ci-dessus - recherche de contenu spécifique
display(item2)

item3 = gis.content.get('edebfe03764b497f90cda5f0bfe727e2') #Le numéro d'identification de l'article
display(item3)
```

<hr>

### Effectuer une analyse
Une fois les couches introduites dans le bloc-notes Jupyter, nous sommes en mesure d'effectuer des types d'analyse similaires que vous vous attendez Ã  trouver dans un logiciel SIG tel qu'ArcGIS. Il existe de nombreux modules contenant de nombreux sous-modules dont peuvent effectuer plusieurs types d'analyses.
<br/>

À l'aide du module arcgis.features, importez le sous-module use_proximity ```from arcgis.features import use_proximity```. Ce sous-module nous permet de '.create_buffers' - zones de distance égale par rapport aux entités. Ici, nous spécifions la couche que nous voulons utiliser, la distance, les unités et le nom en sortie (vous pouvez également spécifier d'autres caractéristiques telles que le champ, le type d'anneau, le type d'extrémité et autres). En spécifiant un nom en sortie, après avoir exécuté la commande de zone tampon, une nouvelle couche sera automatiquement téléchargée dans le portail GEO SAD contenant la nouvelle fonctionnalité que vous venez de créer.
<br/>

```python
buffer_lyr = use_proximity.create_buffers(item1, distances=[1], 
                                          units = "Kilometers", 
                                          output_name='item1_buffer')

display(item1_buffer)
```

Certains utilisateurs préfèrent travailler avec des packages Open Source.  La traduction d'ArcGIS vers spatial Dataframes est simple.
```python
# create a Spatially Enabled DataFrame object
sdf = pd.DataFrame.spatial.from_layer(feature_layer)
```

<hr>

### Mettre à jour les éléments
En obtenant l'élément comme nous l'avons fait similaire à l'exemple ci-dessus, nous pouvons utiliser la fonction '.update' pour mettre à jour l'élément existant dans le portail SAD GEO. Nous pouvons mettre à jour les propriétés des éléments, les données, les vignettes et les métadonnées.
```python
item1_buffer = gis.content.get('c60c7e57bdb846dnbd7c8226c80414d2')
item1_buffer.update(item_properties={'title': 'Enter Title'
									 'tags': 'tag1, tag2, tag3, tag4',
                                     'description': 'Enter description of item'}
```

<hr>

### Visualisez vos données sur une carte interactive

**Exemple : Bibliothèque MatplotLib**
Dans le code ci-dessous, nous créons un objet ax, qui est un tracé de style carte. Nous traçons ensuite notre colonne de changement de données ('Changement de population') sur les axes
```python
import matplotlib.pyplot as plt
ax = sdf.boundary.plot(figsize=(10, 5))
shape.plot(ax=ax, column='Population Change', legend=True)
plt.show()
```

**Exemple : bibliothèque ipyleaflet**
Dans cet exemple, nous utiliserons la bibliothèque 'ipyleaflet' pour créer une carte interactive. Cette carte sera centrée autour de Toronto, en Ontario. Les données utilisées seront décrites ci-dessous.
Commencez par coller ```conda install -c conda-forge ipyleaflet``` vous permettant d'installer des bibliothèques ipyleaflet dans l'environnement Python.
<br/>
Importez les bibliothèques nécessaires.
```python
import ipyleaflet 
from ipyleaflet import *
```
Maintenant que nous avons importé le module ipyleaflet, nous pouvons créer une carte simple en spécifiant la latitude et la longitude de l'emplacement que nous voulons, le niveau de zoom et le fond de carte [(plus de fonds de carte)](https://ipyleaflet.readthedocs.io/en/latest/map_and_basemaps/basemaps.html) (en anglais seulement).  Des contrôles supplémentaires ont été ajoutés, tels que des couches et des mises Ã  l'échelle.
```python
toronto_map = Map(center=[43.69, -79.35], zoom=11, basemap=basemaps.Esri.WorldStreetMap)

toronto_map.add_control(LayersControl(position='topright'))
toronto_map.add_control(ScaleControl(position='bottomleft'))
toronto_map
```
<br/>

## En savoir plus sur ArcGIS API for Python
[La documentation complète de l'API ArGIS peut être trouvée ici](https://developers.arcgis.com/python/) (en anglais seulement)

## En savoir plus sur l'environnement analytique géospatial (EGA) et les services du DAS
[Guide d'aide du GAE](https://statcan.github.io/gae-eag/)
