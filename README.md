# Rapport de TP 1 : Manipulation de HDFS avec les commandes SHELL

**Module :** Big Data
**Étudiant :** Toubali Hanane

---

## 1. Fichiers de configuration du projet

Le cluster a été déployé localement en utilisant Docker Compose. Les fichiers de configuration suivants ont été créés à la racine du projet :

* **docker-compose.yaml** : Ce fichier définit l'infrastructure multi-conteneurs, comprenant un nœud maître (NameNode), cinq nœuds de données (DataNodes), ainsi que le gestionnaire de ressources YARN (ResourceManager et NodeManager).
* **config** : Ce fichier centralise l'ensemble des variables d'environnement et des propriétés de configuration pour le système de fichiers HDFS (notamment l'adresse du NameNode et le facteur de réplication par défaut fixé à 3).

---

## 2. Commandes exécutées (Exercice de Synthèse)

Toutes les manipulations ont été effectuées à l'intérieur du conteneur NameNode après s'y être connecté via la commande système :
docker compose exec namenode bash

Voici la suite séquentielle des commandes HDFS exécutées pour réaliser l'exercice de synthèse :

### Création de l'arborescence sur HDFS
hdfs dfs -mkdir /exercice
hdfs dfs -mkdir /exercice/raw
hdfs dfs -mkdir /exercice/archive
hdfs dfs -mkdir /exercice/export

### Création locale et intégration du jeu de données clients
cat > /tmp/clients.csv << 'EOF'
id_client, nom, ville, pays
1, Ahmed, Casablanca, Maroc
2, Fatima, Rabat, Maroc
3, Youssef, Fes, Maroc
4, Sara, Marrakech, Maroc
EOF

hdfs dfs -put /tmp/clients.csv /exercice/raw/

### Visualisation du contenu du fichier sur HDFS
hdfs dfs -cat /exercice/raw/clients.csv

### Sauvegarde et exportation du fichier
hdfs dfs -cp /exercice/raw/clients.csv /exercice/archive/
hdfs dfs -get /exercice/raw/clients.csv /exercice/export/

### Analyse de la taille et de la structure du fichier
hdfs dfs -du -h /exercice/raw/clients.csv
hdfs fsck /exercice/raw/clients.csv -files -blocks -locations

### Modification dynamique de la réplication à 3
hdfs dfs -setrep -w 3 /exercice/raw/clients.csv

---

## 3. Résultats des commandes HDFS

### Structure finale obtenue (Résultat de hdfs dfs -ls -R /exercice) :
drwxr-xr-x   - root supergroup          0 2026-07-13 21:40 /exercice/archive
-rw-r--r--   3 root supergroup        121 2026-07-13 21:40 /exercice/archive/clients.csv
drwxr-xr-x   - root supergroup          0 2026-07-13 21:41 /exercice/export
drwxr-xr-x   - root supergroup          0 2026-07-13 21:38 /exercice/raw
-rw-r--r--   3 root supergroup        121 2026-07-13 21:38 /exercice/raw/clients.csv

### Analyse d'intégrité du fichier (Résultat de hdfs fsck /exercice/raw/clients.csv -files -blocks -locations) :
Le système indique que le fichier de 121 octets est stocké en un seul bloc (blk_1073741825).
Le facteur de réplication effectif est de 3.
Les blocs répliqués sont distribués et actifs sur trois DataNodes distincts du cluster.
Le statut général retourné est : HEALTHY.

---

## 4. Captures d'écran de validation de l'environnement

### État de santé général via l'interface Web du NameNode (localhost:9870) :


<img width="917" height="426" alt="image" src="https://github.com/user-attachments/assets/7a32650d-d90f-4e76-bfd9-7d3b0164c186" />


### État de fonctionnement des conteneurs sur Docker Desktop :

<img width="920" height="419" alt="image" src="https://github.com/user-attachments/assets/891df78f-c694-453f-97c7-a679df6d0b8b" />


---

## 5. Note technique sur la réplication dans HDFS

La réplication est le mécanisme pilier garantissant la haute disponibilité et la tolérance aux pannes au sein de l'écosystème Hadoop :

* **Principe de distribution** : HDFS découpe chaque fichier volumineux en blocs de taille fixe (128 Mo par défaut). Chaque bloc est copié selon un facteur défini (3 dans notre cas) sur des DataNodes différents.
* **Tolérance aux pannes** : En cas de perte d'un ou plusieurs DataNodes (panne matérielle ou déconnexion réseau), le NameNode redirige de façon transparente les lectures de données vers les répliques saines encore en ligne. Les données restent ainsi continuellement accessibles sans interruption de service.
* **Résilience** : HDFS effectue un contrôle d'intégrité constant et réplique automatiquement les blocs perdus sur d'autres nœuds opérationnels pour maintenir à tout moment le niveau de réplication demandé.
