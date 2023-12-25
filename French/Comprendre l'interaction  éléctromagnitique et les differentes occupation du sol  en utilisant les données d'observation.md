# Comprendre l'interaction  éléctromagnitique et les differentes occupation du sol  en utilisant les données d'observation de la terre 

​													Modou mbaye, email : mbaye@modou.dev  +221 779393741 

> La spectroscopie optique est une technique d'analyse qui permet d'étudier les propriétés de la matière en fonction de la lumière qu'elle émet, **absorbe** ou **réfléchit**. Dans le domaine de l'observation de la Terre, la spectroscopie optique est utilisée pour étudier la composition des surfaces terrestres à partir des données recueillies par des satellites tels que **Sentinel 2**, **Landsat** et **MODIS**.

> Sentinel 2 est un satellite d'observation de la Terre développé par l'Agence spatiale européenne (ESA) dans le cadre du programme Copernicus. Il est équipé d'un capteur multispectral qui mesure la réflexion de la lumière solaire sur la surface terrestre dans différentes longueurs d'onde. Le tableau suivant donne une description des bandes , la resoltion et la valeur de longueur des deux satellites de Sentinel-2A et Sentinel-2B en opposition de phase. 

| **Nom** | **Description** | **Resolution**  (métre) |       **Longueur d’onde**       | **Echelle** |
| :-----: | :-------------: | :---------------------: | :-----------------------------: | :---------: |
|   B1    |    Aerosols     |           60            |  443.9nm (S2A) / 442.3nm (S2B)  |   0.0001    |
|   B2    |      Blue       |           10            |  496.6nm (S2A) / 492.1nm (S2B)  |    -----    |
|   B3    |      Green      |           10            |    560nm (S2A) / 559nm (S2B)    |    -----    |
|   B4    |       Red       |           10            |   664.5nm (S2A) / 665nm (S2B)   |    -----    |
|   B5    |   Red Edge 1    |           20            |  703.9nm (S2A) / 703.8nm (S2B)  |    -----    |
|   B6    |   Red Edge 2    |           20            |  740.2nm (S2A) / 739.1nm (S2B)  |    -----    |
|   B7    |   Red Edge 3    |           20            |  782.5nm (S2A) / 779.7nm (S2B)  |    -----    |
|   B8    |       NIR       |           10            |   835.1nm (S2A) / 833nm (S2B)   |    -----    |
|   B8A   |   Red Edge 4    |           20            |   864.8nm (S2A) / 864nm (S2B)   |    -----    |
|   B9    |   Water vapor   |           60            |   945nm (S2A) / 943.2nm (S2B)   |    -----    |
|   B11   |     SWIR 1      |           20            | 1613.7nm (S2A) / 1610.4nm (S2B) |    -----    |
|   B12   |     SWIR 2      |           20            | 2202.4nm (S2A) / 2185.7nm (S2B) |    -----    |



> Grâce à ces mesures, il est possible d'obtenir des informations sur la composition et la **santé des écosystèmes**, *la qualité de l'eau*, *la couverture des sols*, *les changements d'utilisation des terres*, etc. La combinaison des différentes longueurs d'onde permet de discriminer les différents types de végétation, de détecter la présence de polluants ou de caractériser les propriétés physiques des surfaces terrestres.

> **La spectroscopie optique** utilisée avec les données de Sentinel 2 permet donc d'obtenir des cartes détaillées de la composition des surfaces terrestres à une échelle globale. Ces informations sont essentielles pour de nombreuses applications, telles que la gestion des ressources naturelles, la surveillance de l'environnement, l'agriculture de précision, la cartographie des habitats, etc.

##  Zone d'étude

> Ici nous allons considérer  4  occupations de sol a savoir l'eau sur le lac de guier, le sol nu, zone de culture et la végétation aquatique. L'outil géometrie de Google Earth Engine a ete utilisé pour la creation des classes. vous pouvez integrer autant de classe possible dans une zone  donnees. Pour information le lac de guier se trouve au nord du senegal et il fait l'objet de nombreuses etudes scientifiques notament la qualité de son eau.  



<img src="/Users/modoumbaye/Dev/modou%20Cours%20Gee/Comprendre%20l%27interaction%20%20e%CC%81le%CC%81ctromagnitique%20et%20les%20differentes%20occupation%20du%20sol%20%20en%20utilisant%20les%20donne%CC%81es%20d%27observation.assets/fig1.png" alt="fig1.png" style="zoom:80%;" />

> Apres la creation de ces geometries, nous allons les fusionner en creant une collection et en donnant la nom de la classe d'occupation de sol.  Le code suivant permet de creer cette collection de vecteur avec les attributs respectifs. 

```javascript

var all_pts=ee.FeatureCollection([
	ee.Feature(water,{'type':'Eau'}), 
	ee.Feature(soil,{'type':'Sol nu'}), 
	ee.Feature(cropland,{'type':'Culture'}), 
	ee.Feature(vegAquatic,{'type':'Végétation aquatique'})
	])
```

## traitement satellitaire 

> Ici, nous allons  predefinir  le choix de la visualisation de la carte de base, du centrage de la visualisation autour de la zone d'etude. Le code suivant permet de choisir une carte de base satellitaire (ou hybride) et non openstreemap, de centrer avec un zoom de 14 et la delimitation de la zone d'etude 

```javascript
Map.setOptions('satellite')
Map.centerObject(aoi,14)
Map.addLayer(ee.Image().paint(aoi,2,1))
```

> Le bout de code suivant permet de charger les images Sentinel-2,  en considerant la période de janvier au mois d' avril qui ne concerne que  les cultures irriguées.  Un filtrage de la couverture en nuage ('CLOUDY_PIXEL_PERCENTAGE') a été fait et est fixé  à moins de 10 %  en utilisant la fonction (ee.Filter.lt).  le filtrage ee.Filter.calendarRange permet de considérer que les mois de janvier à avril.  Le second filtrage temporelle permet de considérer uniquement l'annee 2023  des cultures irriguées. 

```javascript

var cloudCover=10
var collection=ee.ImageCollection('COPERNICUS/S2')
         .filter(ee.Filter.calendarRange(1,4,'month'))
         .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE',cloudCover))
         .filterDate('2023-01-01','2023-06-30')
         .filterBounds(aoi)
         

```

> Google Earth Engine (GEE) ouvre plusieurs possibilités de visualisation. Ici nous proposons une des facons la plus usitée dans la communaute des utilisateurs de GEE.  Cela consiste a creer les paramètres des visualisation des donnees en s'appuyant sur une combination fausse couleur en utilisant les bandes B11, B8 et B3.  La cellule suivante donne la bout de code et la figure 2  donne une illustration de la visualisation 



```javascript
var viz={min:[1500,1000,1000],
         max:[6000,6000,6000],
         bands:['B11','B8','B3'], 
         opacity:0.65} 

Map.addLayer(collection,viz,'Fausse Couleur')
  
```

> Cette visualisation permet  aussi de verifier certaines classes comme les zones de cultures et de sol car ces classes peuvent subir un changement suivant une saison.  

<img src="/Users/modoumbaye/Dev/modou%20Cours%20Gee/Comprendre%20l%27interaction%20%20e%CC%81le%CC%81ctromagnitique%20et%20les%20differentes%20occupation%20du%20sol%20%20en%20utilisant%20les%20donne%CC%81es%20d%27observation.assets/fig3.png" alt="fig3.png" style="zoom:80%;" />

##  Profile spectral des différentes occupations de sol 

> pour visualiser la reponse spectrale, GEE donne la possibilié de tracer le graphique de la longueur d'onde de chaque occupation de sol en fonction de la reflectance . la fonction  _ui.Chart.image.regions()_  est utilisée dans ce cas de figure en prenant une moyenne sur les 4  mois  de la série d'image que nous appelons collection  dans le jargon Gee.  Ici, nous aussi définis deux nouvelles variables a savoir bands et wavelenght qui sont utiliseees pour  mieux comprendre le profile spectral des classes d'occupation de sol. Nous avons aussi pris le soin de multiplie les images  0.0001 pour ramener les valeurs de la reflectance entre 0 et 1. 

```javascript
var bands=['B1','B2','B3','B4','B5','B6','B7','B8']
var wavelenght=['444','496','560','664','704','740','782','835']


var spectra= ui.Chart.image.regions({
	image:collection.mean().multiply(0.0001).select(bands),
	regions:all_pts,
	reducer:ee.Reducer.mean(),
	scale:10,
	seriesProperty:'type', 
	xLabels:wavelenght
})

print(spectra)

```

> la Figure ci-dessous montre le profile spectrale de l'occupation de sol (OS) de 4 classes à savoir l'eau,le sol nu, la zone de culture et la zone de végétation aquatique.  il est connu que le sol a  un tres grande valeur de reflectance comparé a d'autres types OS comme la végétation dans le domaine du visible. De meme que l'eau a la plus grande absorbance dans dans du domaine du proche infrarouge, ce qui corresponds a notre observation ici.  Entre la classe végétation  aquatique presente un effect optique   mixte à savoir une forte absorbance de l'eau  dans la fenêtre 700nm à 850nm et une forte reflectance de la végétation  de la même fenêtre optique.

<img src="/Users/modoumbaye/Dev/modou%20Cours%20Gee/Comprendre%20l%27interaction%20%20e%CC%81le%CC%81ctromagnitique%20et%20les%20differentes%20occupation%20du%20sol%20%20en%20utilisant%20les%20donne%CC%81es%20d%27observation.assets/fig4.png" alt="fig4.png" style="zoom:80%;" />

## Analyse de la serie temporelle et la detection de changement de classe 

> d'apres la figure du profile spectrale des differents type d'occupation de sol, il ressort que la bande B8 du proche infrarouge  à la longueur d'onde de  835 nm est  un des meilleurs  candidats  de descriminitation des OS et de detection de changement. Le code suivant  permet de visualiser le changement de classe en regardant le profile temporelle dans le proche infrarouge en tenant compte des propretés optiques de 4 classes  entre janvier et juin 2023.   Le point noir ici symbolise la date de changement de la classe culture a une classe sol-nu correspondant a la date apres recolte des cultures dans cette partie du Sénégal, dans la vallée du fleuve.  

```javascript
var chartNir= ui.Chart.image.seriesByRegion({
		imageCollection:collection,
		regions:all_pts,
		reducer:ee.Reducer.mean(),
		band:'B8',
		scale:10,
		seriesProperty:'type',
}).setOptions({
  vAxis:{title:'Reflectance [a.u]'}, 
  hAxis:{title:"Date"}, 
  title:'la serie temporelle de la Bande B8 des occupations de sols'
})

```



![fig5.png](/Users/modoumbaye/Dev/modou%20Cours%20Gee/Comprendre%20l%27interaction%20%20e%CC%81le%CC%81ctromagnitique%20et%20les%20differentes%20occupation%20du%20sol%20%20en%20utilisant%20les%20donne%CC%81es%20d%27observation.assets/fig5.png)



> l'application de la spectroscopie optique aux données d'observation de la Terre, notamment celles fournies par le satellite Sentinel 2, permet d'obtenir des informations précieuses sur la composition des surfaces terrestres à l'échelle globale. Ces informations sont essentielles pour de nombreuses applications dans les domaines de l'environnement, de l'agriculture et de la cartographie.

