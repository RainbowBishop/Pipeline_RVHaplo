# Workflow RVHaplo

[![forthebadge](http://forthebadge.com/images/badges/built-with-love.svg)](http://forthebadge.com)  [![forthebadge](https://forthebadge.com/images/badges/it-works-why.svg)](http://forthebadge.com)  [![forthebadge](https://forthebadge.com/images/badges/for-sharks.svg)](http://forthebadge.com)

Ce pipeline réalisé en Snakemake permet d'automatiser l'utilisation de l'outil bioinformatique RVHaplo (https://github.com/dhcai21/RVHaplo.git)

Le but de cet outil est de reconstruire les haplotypes viraux à partir de données de séquençage 'long reads'

Il va à partir des fichiers '.fastq' ou '.fasta' issus d'un basecalling à la suite d'un séquençage, réaliser un alignement des reads sur la séquence de référence fournie (mapping). Il va ensuite utiliser le fichier créer au format '.sam' dans l'outil RVHaplo afin d'identifier différents haplotypes viraux présents. A la suite de cela, les haplotypes identifiés seront utilisés dans un alignement multiple avec des séquences de référence d'autres isolats / souches puis un arbre phylogénétique sera construit à partir de cet alignement multiple. 

Il est également possible d'appliquer un filtre de longueur minimum pour les reads (par défaut, pas de filtre)



## Pour commencer

**Première étape** : Se connecter à un noeud du cluster (par exemple le noeud 30)

```
ssh node30
```

**Deuxième étape** : Se déplacer dans le scracth du noeud et y créer un dossier personnel pour y travailler

```
cd /scratch/

mkdir {nom_dossier_perso}

cd {nom_dossier_perso}
```

**Troisième étape** : cloner le répertoire Git dans le dossier récemment crée et s'y déplacer

```
git clone https://github.com/RainbowBishop/Pipeline_RVHaplo.git

cd Pipeline_RVHaplo
```

Une fois situé dans le répertoire cloné, modifier les droits d'exécution du fichier rvhaplo.sh

```
chmod +x rvhaplo.sh
```


### Pré-requis

Pour faire fonctionner le pipeline, il est nécessaire de charger 2 modules sur le cluster
  - Miniconda3
  - python 3.8.12

```
module load system/Miniconda3/1.0

module load system/python/3.8.12
```

Enfin, il faut créer un dossier et y placer le ou les fichiers '.fastq' ou '.fasta' des échantillons séquencés. (Facultatif) : Créer un dossier contenant les fichiers 'sequencing_summary.txt' issus du basecalling

Exemple :

```
mkdir reads_files

(facultatif) mkdir sequencing_summary
```

### Configuration du pipeline

Avant de lancer le pipeline, il est nécessaire de le configurer. Direction le fichier config/config.yaml. 

Soit l'ouvrir directement sur le cluster avec un éditeur (vim, nano, ...) soit le transférer localement pour le modifier

Une fois ce fichier ouvert, la ligne "working_directory" est la plus importante, il est nécessaire d'y indiquer le chemin pour
accéder au répertoire de travail (le répertoire contenant les fichiers pour exécuter le pipeline) et qui, s'il n'a pas été changé,
se nomme Pipeline_RVHaplo (pour connaitre le chemin du dossier dans lequel vous êtes situé, utilisez la commande "pwd" directement dans le terminal).

![screen_git](https://user-images.githubusercontent.com/107557836/187189293-c02761c8-e2e1-405b-b515-2aae8e9d0e7a.png)

Si par exemple vous travaillez dans le dossier "/scratch/{nom_dossier_perso}/Pipeline_RVHaplo/", vous devrez remplacer la ligne définit par défaut par celle-ci, ainsi le pipeline saura où aller chercher les fichiers nécessaires à son éxecution. Ne pas oublier le dernier "/".

Ensuite, vérifier si les noms de vos dossiers correspondent au nom de dossier dans le fichier config.yaml.

Enfin, il est possible de modifier d'autres paramètres dans ce fichier comme le nombre de coeurs utilisé par l'outil RVHaplo (entre 1 et 32, plus le nombre est élevé plus l'outil sera rapide) ou le filtre de longueur minimum à appliquer sur les reads (par défaut, le filtre est de 0 donc l'outil conserve tous les reads, si par exemple vous indiquez 200, les reads possédant moins de 200 bases ne seront pas conservés pour l'analyse)

Conseils : 

  - Regarder la longueur moyenne des reads, peut aider sur le choix du filtre (par exemple on ne va pas appliquer un filtre de longueur minimum de 1000 bases si les reads font en moyenne 200 bases, on risquerait de perdre trop de reads et donc de l'information).
  - Si le jeu de données est consitué de 100.000 - 150.000 reads ou moins, ne pas appliquer de filtre (ou alors mettre un seuil faible).

## Démarrage

La ligne de commande pour lancer le pipeline va nécessiter différents arguments

```
--cores [N] | nombre maximum de coeurs utilisables en parallèles pour les différents outils du pipeline

--snakefile [FILE] | fichier snakemake (si le fichier se nomme Snakemake ou snakemake, argument facultatif)

--configfile [FILE] | fichier de configuration

--use-envmodules | pour que le pipeline charge les modules nécessaires à l'utilisation des différents outils

--use-conda | pour que le pipeline installe les différents environnements conda

--conda-frontend conda | pour spécifier que l'on utilise conda et non mamba pour activer les environnements

(Facultatif) -np | lance en mode "dry-run", le pipeline va juste indiquer toutes les tâches qu'il va faire
```

Lors du premier lancement, je conseille le mode "dry-run" pour voir si le pipeline ne détecte pas d'erreur

Exemple : 

```
snakemake -np --configfile config/config.yaml --use-envmodules --use-conda --conda-frontend conda
```

Et pour un "vrai" lancement :

```
snakemake --cores 16 --configfile config/config.yaml --use-envmodules --use-conda --conda-frontend conda
```


## Résultats

Dans le dossier créer pour contenir les résultats, on trouvera un dossier par échantillon inséré dans le pipeline

A l'intérieur on retrouvera :
  - le fichier contenant les haplotypes reconstruits par RVHaplo avec leur abondance ("rvhaplo_haplotypes.fasta")
  - les fichiers d'alignement multiple ("alignment.fasta" pour les isolats ou souches dont les séquences sont complètes & "alignment_CP.fasta" pour les séquences des isolats au niveau de la CP) 
  - l'arbre phylogénétique construit ("NJ_tree.nwk")

## Citations

### 1. Outil RVHaplo
Dehan Cai, Yanni Sun, Reconstructing viral haplotypes using long reads, Bioinformatics, Volume 38, Issue 8, 15 April 2022, Pages 2127–2134, https://doi.org/10.1093/bioinformatics/btac089

#### E-mail: dhcai2-c@my.cityu.edu.hk
#### Version: V3 (2022-08-02 updated)

### 2. Cluster de calcul i-Trop
The authors acknowledge the ISO 9001 certified IRD itrop HPC (member of the South Green Platform) at IRD montpellier for providing HPC resources that have contributed to the research results reported within this paper. URL: https://bioinfo.ird.fr/- http://www.southgreen.fr
