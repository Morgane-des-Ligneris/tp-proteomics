# TP de Protéomique LYON@BioInfo 2021
## Contexte Biologique
Vous allez utiliser des outils informatiques qui vous permettront d’analyser des données brutes issues d’une analyse Shotgun Proteomics récemment publiée dans le journal Science sous le titre "Real-time visualization of drug resistance acquisition by horizontal gene transfer reveals a new role for AcrAB-TolC multidrug efflux pump".

Les données associées à cette publication sont publiques et accessibles sur la plateforme [PRIDE](https://www.ebi.ac.uk/pride/archive/projects/PXD011286). Le PDF de la publication est [`data/Nolivos_2019.pdf`](data/Nolivos_2019.pdf).

## Mise en place

### Méthodologie

Vous forkerez le présent "repository" pour vous permettre de sauvegarder votre travail.
Vous le clonerez ensuite dans votre espace de travail.
Vous éditerez ce fichier `README.md` pour répondre aux questions dans les encarts prévus à cet effet et inserer les figures que vous aurez générées. Ce "repository" vous appartenant, vous pouvez créer tous les repertoires et fichiers necessaires à la conduite du TP.

### Ressources

Seul le langage Python (v3.X) est requis pour ce travail.
Il vous est conseillé d'installer un environnement virtuel python pour installer les libraries requises independamment de votre systèmes d'exploitation.

* Le systême de gestion de paquets [Conda](https://docs.conda.io/en/latest/) est très pratique et disponible pour la plupat des systèmes d'exploitation. Une version légère suffisante pour nos besoin est téléchargeable [ici](https://docs.conda.io/en/latest/miniconda.html)
* Si vous disposez d'un interpreteur python 3.X installé sur votre systême [virtualenv](https://docs.python.org/3/library/venv.html) est désormais "built-in".

#### Procédure conda

Depuis le repertoire de votre repository Git, installez le package scipy et lancez jupyter.

```sh
$PATH_TO_CONDA_DIR/bin/conda install -c conda-forge scipy notebook
$PATH_TO_CONDA_DIR/bin/jupyter notebook
```

#### Procédure virtualenv

Créer l'environnement virtuel.

```python -m venv MADP_TP1```

Activer l'environnement virtuel et installer les packages.

```
source MADP_TP1/bin/activate.sh
pip install --user ipykernel scipy notebook
```
#### Procédure VM IFB

Une "appliance" IFB a été préparée avec les dépendances Python requises.
Elle est accessible [ici](https://biosphere.france-bioinformatique.fr/catalogue/appliance/160/).
Jupyter vous permettra d'ouvrir des terminaux SHELL et des notebook Python.
Le repertoire racine de Jupyter est `/mnt/mydatalocal/`


#### Intégration des environnements au notebook

Il peut être pratique d'ajouter votre environnement à Jupyter. Cela se réalise depuis l'environnement (conda ou venv) activé.

```
python -m ipykernel install --user --name=MADP_TP1
```


Jupyter est une environnement de type notebook permettant l'execution de code python dans des cellules avec une persitance des variables entre chaque evaluation de cellule. Jupyter fournit nativement le support de la librarie graphique matplotlib.

### Test de l'installation

Dans l'inteface de jupyter, créez un nouveau fichier notebook (*.ipynb) localisé dans votre repertoire git.
Dans la première cellule copiez le code suivant:

```python
%matplotlib nbagg
import matplotlib.pyplot as plt
import numpy as np
from scipy.stats import norm
```

La première ligne sert à activer le rendu graphique, pour tout le fichier notebook. Pour **dessiner des graphiques**, il vous est conseillé de suivre la méthode illustrée par le code suivant que vous pouvez executer dans une deuxième cellule notebook.

```python
fig, ax = plt.subplots()
x = np.linspace(norm.ppf(0.01),
                norm.ppf(0.99), 100)

ax.plot(x, norm.pdf(x),
       'r-', lw=5, alpha=0.6)

ax.plot(x, np.full(len(x), 0.2),
       'b-', lw=1)

fig.show()
```

![alt text](/img/schema1.png)


* Creation des objets `fig`et `ax`
* Ajout successif de graphiques sur la même figure par l'appel à des methodes de l'objet `ax`
* Affichage de la figure complète via `fig.show()`
* Evaluation de la cellule pour visualisation dans la cellule de résultats.

L'affichage dans la cellule de rendu du notebook devrait confirmer la bonne installation des dépendances.

#### On entend par figure un graphique avec des **axes légendés et un titre**.

La documentation matplotlib est bien faite, mais **attention** il vous est demandé, pour la construction des graphiques, **d'ignorer les méthodes de l'objet `plt`** (frequemment proposées sur le net) et d'utiliser uniquement les méthodes de l'[objet Axes](https://matplotlib.org/api/axes_api.html?highlight=axe#plotting). `plt` est un objet global susceptible d'agir simultanement sur tous les graphiques d'un notebook. A ce stade, son utilisation est donc à éviter.

## Données disponibles

### Mesures experimentales

Un fichier `data/TCL_wt1.tsv` contient les données d'abondances differentielles mesurées sur une souche sauvage d'*Escherichia coli* entre deux conditions: avec Tetracycline et en milieu riche. Le contrôle est le milieu riche.

| Accession | Description | Gene Symbol  |   Corrected Abundance ratio (1.53)    | Log2 Corrected Abundance Ratio | Abundance Ratio Adj. P-Value |   -LOG10 Adj.P-val |
| --- | --- | --- | --- | --- | --- | ---|
| [Accesseur Uniprot](https://www.uniprot.org/help/accession_numbers)  | Texte libre | Texte libre  | <img src="https://render.githubusercontent.com/render/math?math=\frac{\text{WildType}_{\text{Tc}}}{\text{WildType}_{\text{rich}}}"> | <img src="https://render.githubusercontent.com/render/math?math=Log_2(\frac{\text{WildType}_{\text{Tc}}}{\text{WildType}_{\text{rich}}})">  | <img src="https://render.githubusercontent.com/render/math?math=\mathbb{R}^%2B"> | <img src="https://render.githubusercontent.com/render/math?math=\mathbb{R}^%2B">  |

Attention certaines valeurs numériques sont manquantes ou erronées, constatez par vous même en parcourant rapidement le fichier.

### Fiches uniprot

Les fiches de toutes les protéines de *E.coli* sont stockées dans un seul document XML `data/uniprot-proteome_UP000000625.xml`. Nous allons extraire de ce fichier les informations dont nous aurons besoin, à l'aide du module de la librarie standard [XML.etree](https://docs.python.org/3/library/xml.etree.elementtree.html#module-xml.etree.ElementTree). Pour vous facilitez la tâche, les exemples suivants vous sont fournis. Prenez le temps de les executer et de les modifier dans un notebook. Vous vous familliariserez ainsi avec la structure du document XML que vous pouvez egalement inspecter dans un navigateur.

```python
from xml.etree.ElementTree import parse, dump
# Parse the E.Coli proteome XML Document
tree = parse('data/uniprot-proteome_UP000000625.xml')
root = tree.getroot()
ns = '{http://uniprot.org/uniprot}' # MANDATORY PREFIX FOR ANY SEARCH within document
# Store all entries aka proteins in a list of xml nodes
proteins = root.findall(ns + 'entry')
# Display the xml subtree of the first protein 
dump(proteins[0])
```

```python
# Find the xml subtree of a protein with accession "P31224"
for entry in proteins:
    accessions = entry.findall(ns+"accession")
    for acc in accessions:
        if acc.text == "P31224":
            dump(entry)
            break
```

Cherchez par exemple le sous arbre de la protéine nommée `DACD_ECOLI`

### Statistiques de termes GO

Les nombres d'occurences de tous les termes GO trouvés dans le protéome de E.coli sont stockés dans le fichier `data/EColiK12_GOcounts.json`. Ces statistiques ont été dérivées du fichier `data/uniprot-proteome_UP000000625.xml`.

## Objectifs

1. Manipuler les données experimentales tabulées
2. Representer graphiquement les données d'abondance
3. Construire la pvalue des fonctions biologiques (termes GO) portées par les protéines surabondantes
4. Visualiser interactivement les pathways plus enrichis.

### Description statistique des Fold Change
La lecture des données au format tabulé est l'occasion de se familliariser avec la [librairie pandas](https://pandas.pydata.org).

##### Lecture de données

###### source:`data/TCL_wt1.tsv`

La fonction `read_csv` accepte différents [arguments](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html) de format de données très utiles.

```python
df = pandas.read_csv('data/TCL_wt1.tsv',sep='\t', header=0)
```

Quel est le type de l'objet `df`?
```
type(df)
pandas.core.frame.DataFrame

C'est un data frame
```

##### Descriptions d'une table de données
Que permettent les méthodes suivantes?
###### df.shape
```
#Tuple of array dimensions.
(2024, 7)
```
###### df.head()
```
df.head()#The first n rows of the caller object. # interprété en HTML, plus qu'un tableau .txt

	Accession	Description	Gene Symbol	Corrected Abundance ratio (1.53)	Log2 Corrected Abundance Ratio	Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)	-LOG10 Adj.P-val
0	P75936	Basal-body rod modification protein FlgD OS=Es...	flgD	0.075816993	-3.721334942	0.000055	4.260067469
1	P76231	Uncharacterized protein YeaC OS=Escherichia co...	yeaC	0.092810458	-3.429568818	0.000351	3.45462743
2	P0A8S9	Flagellar transcriptional regulator FlhD OS=Es...	flhD	0.102614379	-3.284695189	0.000027	4.571899347
3	P0CE48	Elongation factor Tu 2 OS=Escherichia coli (st...	tufB	#VALEUR!	#VALEUR!	NaN	#VALEUR!
4	P05706	PTS system glucitol/sorbitol-specific EIIA com...	srlB	0.108496732	-3.204276506	0.019963	1.699767669

```
###### df.tail()
```
df.tail() #Returns the last n rows.

Accession	Description	Gene Symbol	Corrected Abundance ratio (1.53)	Log2 Corrected Abundance Ratio	Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)	-LOG10 Adj.P-val
2019	P24240	6-phospho-beta-glucosidase AscB OS=Escherichia...	ascB	#VALEUR!	#VALEUR!	NaN	#VALEUR!
2020	P0A917	Outer membrane protein X OS=Escherichia coli (...	ompX	1.579738562	0.65968582	0.002226	2.652390664
2021	P02931	Outer membrane protein F OS=Escherichia coli (...	ompF	1.754901961	0.811390435	0.000068	4.16495627
2022	P0AB40	Multiple stress resistance protein BhsA OS=Esc...	bhsA	1.798039216	0.846424487	0.035928	1.444561032
2023	P76042	Putative ABC transporter periplasmic-binding p...	ycjN	#VALEUR!	#VALEUR!	NaN	#VALEUR!

```

###### df.columns
```
df.columns #The column labels of the DataFrame.
Index(['Accession', 'Description', 'Gene Symbol',
       'Corrected Abundance ratio (1.53)', 'Log2 Corrected Abundance Ratio',
       'Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)',
       '-LOG10 Adj.P-val'],
      dtype='object')
```
###### df.dtypes
```
df.dtypes #This returns a Series with the data type of each column. (Columns with mixed types are stored with the object dtype.)
Accession                                                        object
Description                                                      object
Gene Symbol                                                      object
Corrected Abundance ratio (1.53)                                 object
Log2 Corrected Abundance Ratio                                   object
Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)    float64
-LOG10 Adj.P-val                                                 object
dtype: object
```
###### df.info
```
df.info #Print a concise summary of a DataFrame.
<bound method DataFrame.info of      Accession                                        Description Gene Symbol  \
0       P75936  Basal-body rod modification protein FlgD OS=Es...        flgD   
1       P76231  Uncharacterized protein YeaC OS=Escherichia co...        yeaC   
2       P0A8S9  Flagellar transcriptional regulator FlhD OS=Es...        flhD   
3       P0CE48  Elongation factor Tu 2 OS=Escherichia coli (st...        tufB   
4       P05706  PTS system glucitol/sorbitol-specific EIIA com...        srlB   
...        ...                                                ...         ...   
2019    P24240  6-phospho-beta-glucosidase AscB OS=Escherichia...        ascB   
2020    P0A917  Outer membrane protein X OS=Escherichia coli (...        ompX   
2021    P02931  Outer membrane protein F OS=Escherichia coli (...        ompF   
2022    P0AB40  Multiple stress resistance protein BhsA OS=Esc...        bhsA   
2023    P76042  Putative ABC transporter periplasmic-binding p...        ycjN   

     Corrected Abundance ratio (1.53) Log2 Corrected Abundance Ratio  \
0                         0.075816993                   -3.721334942   
1                         0.092810458                   -3.429568818   
2                         0.102614379                   -3.284695189   
3                            #VALEUR!                       #VALEUR!   
4                         0.108496732                   -3.204276506   
...                               ...                            ...   
2019                         #VALEUR!                       #VALEUR!   
2020                      1.579738562                     0.65968582   
2021                      1.754901961                    0.811390435   
2022                      1.798039216                    0.846424487   
2023                         #VALEUR!                       #VALEUR!   

      Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)  \
0                                              0.000055              
1                                              0.000351              
2                                              0.000027              
3                                                   NaN              
4                                              0.019963              
...                                                 ...              
2019                                                NaN              
2020                                           0.002226              
2021                                           0.000068              
2022                                           0.035928              
2023                                                NaN              

     -LOG10 Adj.P-val  
0         4.260067469  
1          3.45462743  
2         4.571899347  
3            #VALEUR!  
4         1.699767669  
...               ...  
2019         #VALEUR!  
2020      2.652390664  
2021       4.16495627  
2022      1.444561032  
2023         #VALEUR!  

[2024 rows x 7 columns]>
```
###### df.describe()
```
df.describe() #Generate descriptive statistics.
	Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)
count	1.750000e+03
mean	8.238311e-01
std	3.506349e-01
min	1.034030e-08
25%	1.000000e+00
50%	1.000000e+00
75%	1.000000e+00
max	1.000000e+00

```
###### df.dropna()
```
df.dropna() #Remove missing values. #retourne une nouvelle data frame avec des lignes qui ont sauté (celle où il manquait des infos)

Accession	Description	Gene Symbol	Corrected Abundance ratio (1.53)	Log2 Corrected Abundance Ratio	Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)	-LOG10 Adj.P-val
0	P75936	Basal-body rod modification protein FlgD OS=Es...	flgD	0.075816993	-3.721334942	0.000055	4.260067469
1	P76231	Uncharacterized protein YeaC OS=Escherichia co...	yeaC	0.092810458	-3.429568818	0.000351	3.45462743
2	P0A8S9	Flagellar transcriptional regulator FlhD OS=Es...	flhD	0.102614379	-3.284695189	0.000027	4.571899347
4	P05706	PTS system glucitol/sorbitol-specific EIIA com...	srlB	0.108496732	-3.204276506	0.019963	1.699767669
5	P29744	Flagellar hook-associated protein 3 OS=Escheri...	flgL	0.124183007	-3.009460329	0.036746	1.434786589
...	...	...	...	...	...	...	...
2011	P77330	Prophage lipoprotein Bor homolog OS=Escherichi...	borD	1.535947712	0.619129104	0.310725	0.507623276
2016	P02930	Outer membrane protein TolC OS=Escherichia col...	tolC	1.552287582	0.634395861	0.013373	1.873756665
2020	P0A917	Outer membrane protein X OS=Escherichia coli (...	ompX	1.579738562	0.65968582	0.002226	2.652390664
2021	P02931	Outer membrane protein F OS=Escherichia coli (...	ompF	1.754901961	0.811390435	0.000068	4.16495627
2022	P0AB40	Multiple stress resistance protein BhsA OS=Esc...	bhsA	1.798039216	0.846424487	0.035928	1.444561032
1746 rows × 7 columns
```

##### Accès aux éléments d'une table de données

```python
values = df[['Description', 'Gene Symbol']]
```

Quel est le type de `values` ?

```
type(values)
pandas.core.frame.DataFrame

values.shape
(2024, 2)
```

Verifiez si certaines méthodes de `DataFrame` lui sont applicables.
Ce type supporte l'accès par indice et les slice `[a:b]`

##### Accès indicé

On peut accéder aux valeurs du DataFrame via des indices ou plages d'indice. La structure se comporte alors comme une matrice. La cellule en haut et à gauche est de coordonnées (0,0).
Il y a différentes manières de le faire, l'utilisation de `.iloc[slice_ligne,slice_colonne]` constitue une des solutions les plus simples. N'oublions pas que shape permet d'obtenir les dimensions (lignes et colonnes) du DataFrame.
###### Acceder aux cinq premières lignes de toutes les colonnes
```python
df.iloc[0:5,]
	Accession	Description	Gene Symbol	Corrected Abundance ratio (1.53)	Log2 Corrected Abundance Ratio	Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)	-LOG10 Adj.P-val
0	P75936	Basal-body rod modification protein FlgD OS=Es...	flgD	0.075816993	-3.721334942	0.000055	4.260067469
1	P76231	Uncharacterized protein YeaC OS=Escherichia co...	yeaC	0.092810458	-3.429568818	0.000351	3.45462743
2	P0A8S9	Flagellar transcriptional regulator FlhD OS=Es...	flhD	0.102614379	-3.284695189	0.000027	4.571899347
3	P0CE48	Elongation factor Tu 2 OS=Escherichia coli (st...	tufB	#VALEUR!	#VALEUR!	NaN	#VALEUR!
4	P05706	PTS system glucitol/sorbitol-specific EIIA com...	srlB	0.108496732	-3.204276506	0.019963	1.699767669

```

###### Acceder à toutes les lignes de la dernière colonne
```python
df.iloc[:,6:7]

-LOG10 Adj.P-val
0	4.260067469
1	3.45462743
2	4.571899347
3	#VALEUR!
4	1.699767669
...	...
2019	#VALEUR!
2020	2.652390664
2021	4.16495627
2022	1.444561032
2023	#VALEUR!
2024 rows × 1 columns

```

###### Acceder aux cinq premières lignes des colonnes 0, 2 et 3
```python
df.iloc[0:5,[0,2,3]]
#df.iloc[[0,1,2,3,4],[0,2,3]]
	Accession	Gene Symbol	Corrected Abundance ratio (1.53)
0	P75936	flgD	0.075816993
1	P76231	yeaC	0.092810458
2	P0A8S9	flhD	0.102614379
3	P0CE48	tufB	#VALEUR!
4	P05706	srlB	0.108496732

```

##### Conversion de type

Le type des valeurs d'une colonne peut être spécifiée:

* à la lecture

```python
# rajouter l'argument na_values='#VALEUR!'
pandas.read_csv('data/TCL_wt1.tsv', sep="\t", na_values='#VALEUR!, dtype = {'Accession': str, 'Description': str, 'Gene Symbol': str, 
                                                 'Corrected Abundance ratio (1.53)': np.float,  'Log2 Corrected Abundance Ratio': np.float, 
                                                 'Abundance Ratio Adj. P-Value: (127. T3 Tc WT) / (126. T0 WT)': np.float, '-LOG10 Adj.P-val': np.float})
```

* modifiée à la volée

```python
df = df..dropna().astype({'Log2 Corrected Abundance Ratio': float, '-LOG10 Adj.P-val': float } )
```

##### Selection avec contraintes
La méthode `loc` permet de selectionner toutes les lignes/colonnes respectant certaines contraintes

* Contraintes de valeurs continues

```python
df.loc[(df['-LOG10 Adj.P-val'] > 0 )  & (df['Log2 Corrected Abundance Ratio'] > 0.0 ) ]
```

* Contraintes de valeurs discrètes

```python
df.loc[ df['Gene Symbol'].isin(['fadR', 'arcA'] ) ]
```


#### Appliquons ces outils à l'analyse de données protéomique

##### 1. Charger le contenu du fichier `data/TCL_wt1.tsv` dans un notebook en eliminant les lignes porteuses de valeurs numériques aberrantes

##### 2. Representez par un histogramme les valeurs de `Log2 Corrected Abundance Ratio`

##### 3. A partir de cette échantillon de ratio d'abondance,  estimez la moyenne <img src="https://render.githubusercontent.com/render/math?math=\mu"> et l'ecart-type <img src="https://render.githubusercontent.com/render/math?math=\sigma"> d'une loi normale.
```
Log2(Abundance Ratio)  
    X barre => mu 
    S2 => S_2 ≈ sigma^2

```

##### 4. Superposez la densité de probabilité de cette loi sur l'histogramme. Attention, la densité de probabilité devra être mis à l'echelle de l'histogramme (cf ci-dessous)

S_2 = n / (n-1) Var(Log2(Abundance Ratio))
    X barre => mu 



```python
fig, ax = plt.subplots()
hist = ax.hist(_, bins=100) # draw histogram
x = np.linspace(min(_), max(_), 100) # generate PDF domain points
dx = hist[1][1] - hist[1][0] # Get single value bar height
scale = len(_)*dx # scale accordingly
ax.plot(x, norm.pdf(x, mu, sqrt(S_2))*scale) # compute theoritical PDF and draw it
```

![Histogramme à inserez ici](histogram_log2FC.png "Title")

##### 5. Quelles remarques peut-on faire à l'observation de l'histogramme et de loi théorique?

```


```

#### Construction d'un volcano plot

##### A l'aide de la méthode [scatter](https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.axes.Axes.scatter.html) representer <img src="https://render.githubusercontent.com/render/math?math=-\text{Log}_{10}({\text{p-value}}) = f(\text{Log}_2(\text{abundance ratio}))">

##### Matérialisez le quadrant des protéines surabondantes, par deux droites ou un rectangle
Sont condidérées comme surabondantes les proteines remplissant ces deux critères:

* <img src="https://render.githubusercontent.com/render/math?math=\text{Log}_2(\text{abundance ratio})\gt\mu%2B\sigma">  
* <img src="https://render.githubusercontent.com/render/math?math=\text{p-value}>0.001">

![Volcano plot + quadrant à inserez ici](histogram_log2FC.png "Title")

### Analyse Fonctionelle de pathway

Nous allons implementer une approche ORA (Over Representation Analysis) naive.

##### 1. Retrouver les entrées du fichier TSV des protéines surabondates

Quelles sont leurs identifiants UNIPROT ?
``` 



```

#### 2. Lister les termes GO portés par ces protéines surabondates

Les `entry` du fichier `data/uniprot-proteome_UP000000625.xml` présentent des balises de ce type:

```xml
<dbReference type="GO" id="GO:0005737">
<property type="term" value="C:cytoplasm"/>
<property type="evidence" value="ECO:0000501"/>
<property type="project" value="UniProtKB-SubCell"/>
</dbReference>
```

A ce stade, on se contentera des identifiants GO (eg `GO:0005737`). Vous pouvez faire un [set](https://docs.python.org/3.8/library/stdtypes.html#set) de cette liste d'identifiants GO pour en éliminer rapidement la redondance.

#### 3. Obtention des paramètres du modèle

Nous evaluerons la significativité de la présence de tous les termes GO portés par les protéines surabondantes à l'aide d'un modèle hypergéometrique.

Si k protéines surabondantes porte un terme GO, la pvalue de ce terme sera équivalente à <img src="https://render.githubusercontent.com/render/math?math=P(X\ge k), X \sim H(k,K,n,N)">.
Completer le tableau ci-dessous avec les quantités vous  semblant adéquates

| Symboles | Paramètres | Quantités Biologiques |
| --- | --- | --- |
| k | nombre de succès observés| |
| K | nombre de succès possibles| |
| n | nombre d'observations| |
| N | nombre d'elements observables| |

#### 4. Calcul de l'enrichissement en fonction biologiques

A l'aide du contenu de `data/EColiK12_GOcounts.json` parametrez la loi hypergeometrique et calculez la pvalue
de chaque terme GO portés par les protéines surabondantes. Vous reporterez ces données dans le tableau ci-dessous

| identifiant GO | définition | occurence | pvalue|
|---|---|---|---|
|   |   |   |   |

Quelle interpretation biologique faites-vous de cet enrichissement en termes GO spécifiques ?

```




```


