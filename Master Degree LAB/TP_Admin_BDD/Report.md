# Administration de base de données :  Rapport de TP
# KENEY MESSAN DAVID 
# Université Clermont Auvergne
### 2018-2019 





# TP 1  :  Analyse d’une instance Oracle existante


1. Connectez-vous à l’instance Oracle existante.
```sql
sqlplus / as sysdba
startup
```
2. Quel est le nom de l’instance à laquelle vous êtes connectés ?

```sql
select instance_name from v$instance;
```
> orcl



3. Identifiez le nom de la base de données, le nom de l’instance et la taille des blocs de la base de données


```sql
select database_name from v$databse; 
or
show parameter db_name;
```
> ORCL

```sql
show parameter instance_name;
```
> orcl

```sql
show parameter db_block_size;
```
> 8192


4. Énumérez le nom et la taille des fichiers de données, des fichiers de reprises et le nom des fichiers de contrôle


```sql
select name,blocks,block_size from v$datafile;
```
| NAME | BLOCKS | BLOCK_SIZE |
|------|--------|------------|
|  /u01/app/oracle/oradata/orcl/system01.dbf    |   87040     |      8192      |
|    /u01/app/oracle/oradata/orcl/sysaux01.dbf  |     67840   |8192|
|    /u01/app/oracle/oradata/orcl/undotbs01.dbf  |     13440   |     8192       |
|  /u01/app/oracle/oradata/orcl/users01.dbf    |   640     |      8192      |
|   /u01/app/oracle/oradata/orcl/example01.dbf   |    12800    |        8192    |


```sql
select * from v$controlfile;
```
| NAME | BLOCKS | BLOCK_SIZE |
|------|--------|------------|
|  u01/app/oracle/oradata/orcl/control01.ctl  |   16384     |      594      |
|    /u01/app/oracle/flash_recovery_area/orcl/control02.ctl |     16384   |594|
|    /u01/app/oracle/oradata/orcl/undotbs01.dbf  |     13440   |     8192       |

```sql
select * from v$logfile;
```
>/u01/app/oracle/oradata/orcl/redo03.log
/u01/app/oracle/oradata/orcl/redo02.log
/u01/app/oracle/oradata/orcl/redo01.log

5. Quelles sont les options installées ?

```sql
select * from v$option where value='TRUE';
```
6. Affichez le numéro de version.

```sql
select version from V$version;
```
> 11.2.0.1.0

7. Donnez le nombre maximum de processus utilisateur pouvant se connecter simultanément à l’instance.
```sql
select value from v$parameter where name='processes';
```
> 150
8. Quel est le nombre de vues dynamiques ?

```sql
select count(*) from V$fixed_table where (name LIKE 'V$%');
```
> 0
9. Tentez de modifier la taille du bloc de la base de données, i.e. arréter la base de données, modifier le paramètre db_block_size (doubler sa taille), redémarrer la base de données et observer ce qui se passe.

```sh
shutdown
cd /u01/app/oracle/admin/orcl/pfile
cp init.ora.94201816856  init.ora
gedit init.ora
```
>On double la taille du block à 16384 dans le fichier (manuellement)
```sh
syqplus / as sysdba
shutdown immediate
startup pfile=/uo1/app/oracle/admin/orcl/pfile/init.ora
startup nomount pfile=/uo1/app/oracle/admin/mysint/pfile/init.ora
```
> DB_BLOCK_SIZE must be 8192 to mount this database (not 16384)

10. Énumérez les paramètres d’initialisation par défaut.

```sql
select name from v$parameter where isdefault='TRUE';
```
11. Connectez vous sous SCOTT/TIGER et insérez des lignes dans la table EMP. Ouvrez une seconde session et essayez d’arrêter la base de données en mode normal et ensuite en mode shutdown immediate, expliquez ce qu’il se passe pour chaque cas. Rédémarrer l’instance. Ouvrez une nouvelle session sqlplus en tant que SCOTT/TIGER et vérifier ce qui s’est passé avec l’employé ’toto’ que vous avez inséré précédemment.
```sql
connect scott/toger ou sqlplus scott/tiger
INSERT INTO EMP(EMPNO,ENAME,JOB,MGR,HIREDATE,SAL,COMM,DEPTNO) VALUES ( 999,'KENEY','CEO',1500,'10-DEC-2018',1500,5,20); 
```


>shutdown normal :  oracle se met en mode attente jusqu'a ce que l'utilisateur se deconnecte.
shutdown immediate : oracle déconnecte scott puis proccède à l'arrêt, l'arrêt est  plus rapide
shutdown transactional : oracle attend la validation (commit ou rollback ) de la transaction avant l' arrêt

### Création d’une base de données
1. Dans le répertoire /u01/app/oracle/admin, créer un nouveau sous-répertoire myinst.
 ```sh
cd  /u01/app/oracle/admin
mkdir myinst
```
2. Dans les répertoires /u01/app/oracle/oradata/ et /u01/app/oracle/flash_recovery_area/, créer deux nouveaux sous-répertoires appelés myinst.
 ```sh
cd  /u01/app/oracle/oradata/
mkdir myinst
cd  /u01/app/oracle/flash_recovery_area/
mkdir myinst
```
3. Créér les sous-répertoires suivants dans le répertoire /u01/app/oracle/admin/myinst : adump, dpdump et pfile.
 ```sh
cd  /u01/app/oracle/admin/myinst 
mkdir adump dpdump pfile
```
4. Créer un nouveau fichier de paramètre, par exemple initmyinst.ora dans le répertoire pfile (e.g. copier un fichier existant puis le personnaliser ou bien connectez-vous à l’instance orcl et généré un fichier pfile à partir de son fichier spfile).

 ```sh
cd  /u01/app/oracle/admin/myinst 
mkdir adump dpdump pfile
```
5. Récupérer le script creationBD.sql, disponible au niveau de l’ent, et le modifier pour permettre la création d’une base de données avec la configuration suivante :
— Nom de la base de données et nom de l’instance myinst
— un fichier de contrôle appelé control01.ctl placé dans le répertoire /u01/app/oracle/oradata/myinst,
— deux groupes de fichiers de reprise avec chacun 1 membre de 150K appelé myinst_log1a.log et myinst_log2a.log placé dans le répertoire /u01/app/oracle/oradata/myinst.
— un nombre maximum de 5 groupes de fichiers log et 5 membres fichiers log dans chaque groupe.
— un fichier de données de 20Mo appelé system01myinst.dbf et placé dans le répertoire   /u01/app/oracle/oradata/myinst,
— un maximum de 30 fichiers de données pour la base de données
— le jeu de caractères US7ASCII

 ```sql
CREATE DATABASE myinst
USER SYS IDENTIFIED BY oracle
USER SYSTEM IDENTIFIED BY oracle
LOGFILE
GROUP 10 '/u01/app/oracle/oradata/myinst/myinst_log1a.log' SIZE 50M,
GROUP 20 '/u01/app/oracle/oradata/myinst/myinst_log2a.log' SIZE 50M,
MAXLOGFILES 5
MAXLOGMEMBERS 5
MAXLOGHISTORY 1
MAXDATAFILES 30
MAXINSTANCES 1
CHARACTER SET US7ASCII
NATIONAL CHARACTER SET AL16UTF16
DATAFILE '/u01/app/oracle/oradata/myinst/system01myinst.dbf' SIZE 100M
REUSE
EXTENT MANAGEMENT LOCAL
SYSAUX DATAFILE '/u01/app/oracle/oradata/myinst/sysaux01.dbf' SIZE
325M REUSE
DEFAULT TEMPORARY TABLESPACE tempts1
TEMPFILE '/u01/app/oracle/oradata/myinst/temp01.dbf'
SIZE 20M REUSE
UNDO TABLESPACE undotbs1
DATAFILE '/u01/app/oracle/oradata/myinst/undotbs01.dbf' SIZE 200M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED;
```
6. Démarrez l’instance en mode nomount et exécutez le script.
```sh
shutdown immediate
startup nomount pfile=/u01/app/oracle/admin/myinst/pfile/initmyinst.ora
start /home/oracle/Downloads/TP_ADMIN_BDD/sujet_tp/TP1/creationBD.sql
```

7. Après la création, montez la base de données puis vérifiez son état et assurez-vous que les fichiers de la base de données ont été créés.
```sh
select name,blocks from v$datafile;
```
| NAME | BLOCKS | 
|------|--------|
|  /u01/app/oracle/oradata/myinst/system01myinst.dbf |   12800     |
|   /u01/app/oracle/oradata/myinst/sysaux01.dbf |     41600   |
|/u01/app/oracle/oradata/myinst/undotbs01.db| 25600 | 

8. Tentez d’afficher le nom des utilisateurs de la base de données ? Que se passe-t-il ?
```sql
select user from dba_users;;
```
| NAME  | 
|------|
| SYS |
| SYS |
| SYS |

### Création de vues du dictionnaire de données et Création des packages Standard
> Questions 1 à 4
```sql
startup  pfile=/u01/app/oracle/admin/myinst/pfile/initmyinst.ora
start /u01/app/oracle/product/11.2.0.4/db_1/rdbms/admin/catalog.sql
start /u01/app/oracle/product/11.2.0.4/db_1/rdbms/admin/catproc.sql
start /u01/app/oracle/product/11.2.0.4/db_1/rdbms/admin/utlsampl.sql
```
5. Vérifier que l’utilisateur Scott/tiger a été créé ainsi que la base de données exemple associée.
> Oui, l'utitlisiteur et la base  existe

# TP 2  : Utilisation des outils d'administration mode ligne


1. Connectez-vous en tant que SYS à l’instance oracle myinst que vous avez créé lors du TP précédent. Vérifier que vous êtes connectés à la bonne instance.
```sql
startup pfile=/u01/app/oracle/admin/myinst/pfile/initmyinst.ora upgrade
show parameter instance_name;
```
| NAME  | TYPE  | VALUE  | 
|------|------|------|
|instance_name|string|myinst|
2. Quelle est la taille du buffer cache de données ?
```sql
show parameter db_cache_size;
```
| NAME  | TYPE  | VALUE  | 
|------|------|------|
|db_cache_size |big integer |0|
3. Quelle est la taille de la SGA (zone mémoire globale du système) ?
```sql
select * from v$sgainfo;
```
| NAME  | BYTES  | RES  | 
|------|------|------|
|Maximum SGA Size   |1586708480 |No|
> 12 rows selected.
4. Lister les colonnes owner, table_name, tablespace_name de la vue dba_tables du dictionnaire de données pour les tables et les tablespaces dont le propriétaire est l’utilisateur SCOTT.
```sql
SELECT owner,table_name,tablespace_name FROM dba_tables;
```
| OWNER  | TABLE_NAME  | TABLESPACE_NAME  | 
|------|------|------|
|SYS | WRI$_ADV_SQLA_MAP |SYSAUX|
|SYS |WRI$_ADV_SQLA_TABLES |SYSAUX|
|SYS | WRI$_ADV_SQLA_FAKE_REG |SYSAUX|
> 728 rows selected

# Mise à jour du fichier de contrôle
1. Sauvegarder le fichier de contrôle (il faut générer l’ordre SQL qui permet de le recréer)
```sql
ALTER DATABASE BACKUP CONTROLFILE TO TRACE;
```
> database altered
```sql
select value from v$diag_info where name ="default trace file";
```
>le fichier se trouve : /u01/app/oracle/diag/rdbms/orcl/orcl/trace

2. Vérifier que votre sauvegarde s’est bien passée
> Elle a bien été sauvegardé

3. Où sont placés les fichiers de contrôle existants et quels sont leurs noms ?
> Les fichiers se trouvent dans /u01/app/oracle/oradata/myinst

4. Essayez de démarrer la base de données sans fichier de contrôle. Pour cela, il faut créer une copie /u01/app/oracle/admin/myinst/pfile/initmyinst2.ora du fichier de paramètres /u01/app/oracle/admin/myinst/pfile/initmyinst.ora. Commenter ensuite la ligne control_files dans le fichier /u01/app/oracle/admin/myinst/pfile/ et démarrer la base de données avec ce fichier de paramètres. Que se passe-t-il ?
```sh
cd  /u01/app/oracle/admin/myinst/pfile/
cp initmyinst.ora  initmyinst2.ora
```

>Apres modification de la ligne control files on exécute 
```sh
startup pfile=/u01/app/oracle/admin/myinst/pfile/initmyinst2.ora
```
> ORA-00205: error in identifying control file, check alert log for more info
5. Pour répondre à cette question, utiliser le fichier de contrôle initmyinst2.ora. Multiplexer les fichiers de contrôle existant en utilisant le répertoire DISK2 dans le répertoire $HOME/adminbdtp2/ et nommer le nouveau fichier de contrôle control03.ctl. Assurez vous que le serveur Oracle peut y écrire. Confirmer que tous fichiers de contrôle sont utilisés.
>Avant de faire cette manipulation il faut fermer le serveur : shutdown immediate
 puis renseigner dans  initmyinst2.ora
control_files=("/u01/app/oracle/oradata/myinst/control01.ctl", "/u01/app/oracle/flash_recovery_area/myinst/control03.ctl")

6. Quel est le nombre maximum de fichiers de données que vous pouvez créer dans la base de données ? et quel est le nombre actuel de fichiers de données de la base de données ?
```sql
show parameter db_files;
startup pfile=/u01/app/oracle/admin/myinst/pfile/initmyinst.ora
select records_total,record_used from v$controlrecord_section where type='DATAFILE';
```

7. Supprimer le fichier de contrôle qui se trouve sur le DISK2
> Fait


# Mise à jour des fichiers de reprise
1. Énumérer le nombre et l’emplacement des fichiers de reprise existants. Afficher le nombre de groupes de fichiers de reprise et le nombre de membres par group que votre base de données contient.
```sql
select count(member) from v$logfile;
```
> 3
```sql
select member from v$logfile;
```
>/u01/app/oracle/oradata/myinst/myinst_log1a.log
/u01/app/oracle/oradata/myinst/myinst_log2a.log
/u01/app/oracle/flash_recovery_area/MYINST/onlinelog/o1_mf_1_fx0j1chs_.log
```sql
select count( distinct group#   ) from v$logfile;
```
> 3
```sql
select group# , members from v$log;
```
| GROUP#  | MEMBERS  | 
|------|------|
|1 | 1 |
|10 | 1 |
|20 | 1 |
2. Dans quel mode de base de données votre base est-elle configurée ? L’archivage est-il activé ?
```sql
archive log list;
```
>Total System Global Area 1586708480 bytes
Fixed Size		    2213736 bytes
Variable Size		  922749080 bytes
Database Buffers	  654311424 bytes
Redo Buffers		    7434240 bytes
Database mounted.
Database opened.
SQL> archive log list
Database log mode	       No Archive Mode
Automatic archival	       Disabled
Archive destination	       USE_DB_RECOVERY_FILE_DEST
Oldest online log sequence     21
Current log sequence


3. Quel est le groupe courant actuel ?
```sql
select group# from v$log where status ="CURRENT";
```
| GROUP#  | STATUS  | 
|------|------|
|10	| CURRENT |
4. Modifier le groupe courant et vérifier que votre modification a été prise en compte.
```sql
ALTER SYSTEM SWITCH LOGFILE;
```

5. Ajouter un membre de reprise à chaque groupe dans votre base de données. Le faire dans le même répertoire avec les conventions suivantes : si le groupe 1 possède un fichier rlog1a.log, ajouter un membre rlog1b.log. Vérifier le résultat.
```sql
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/oradata/myinst/myinst_log1A.log' TO GROUP  10;
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/oradata/myinst/myinst_log2B.log' TO GROUP 20;
```
>Database altered.
6. Créer un nouveau groupe de fichiers de reprise dans le répertoire DISK4 et vérifier son existence.
```sql
ALTER DATABASE ADD LOGFILE GROUP 21 '/u01/app/oracle/oradata/myinst/myinst_log3B.log'  SIZE 10M;
```
>Database altered.
7. Déplacer les membres rlog1b.log et rlog2b.log dans le répertoire DISK4.
> Question faite manuellement
8. Supprimer le groupe de reprise de la question 4.
```sql
ALTER DATABASE DROP LOGFILE GROUP 10;
ALTER DATABASE DROP LOGFILE GROUP 20;
```
9. Supprimer les fichiers de reprise qui se trouvent dans le DISK2
> Question faite manuellement

# TP3 :  Gestion des fichiers deonnées et des tablespaces 

### Gestion des fichiers de données et des tablespaces


1. Dans le répertoire /u01/app/oracle/oradata/myinst/, créer les sous répertoires suivants : Disk1, Disk2, Disk3 et Disk4. 
```sh
cd  /u01/app/oracle/oradata/myinst/
mkdir Disk1 Disk2 Disk3 Disk4
```

2. Connectez vous  en tant que SYS  à l’instance oracle myinst que  vous avez  créé  lors du TP précédent. Vérifier que vous êtes connectés à la bonne instance. Si ce n’est pas la bonne instance  alors il faut arrêter l’instance courante et redémarrer l’instance 
```sql
startup upgrade pfile=/u01/app/oracle/admin/myinst/pfile/initmyinst.ora
```

3. Créer des tablespaces permanents avec les noms et le type de stockage suivants :

- DATA01 (taille de 2Mo)
```sql
CREATE TABLESPACE DATA01 DATAFILE '/u01/app/oracle/oradata/myinst/Disk4/data01.dbf' SIZE 2M;
```
>Tablespace created.


- TEMP pour les segments temporaires (activation de la non augmentation de la taille d’extent). 
```sql
CREATE TEMPORARY TABLESPACE TEMP TEMPFILE '/u01/app/oracle/oradata/myinst/Disk3/temp02.dbf'  SIZE 5M AUTOEXTEND OFF;
```
>Tablespace created.

- INDX01 pour les index avec stockage par défaut (activez l’extension automatique de 500 K si des extents supplémentaires sont requis). 
```sql
CREATE TABLESPACE INDX01 DATAFILE '/u01/app/oracle/oradata/myinst/Disk3/indx01.dbf' SIZE 3M AUTOEXTEND ON NEXT 500K;
```
>Tablespace created.

- RONLY pour les tables en lecture seule avec stockage par défaut create tablespace RONLY datafile ’/u01/app/oracle/oradata/myinst/Disk1/ronly01.dbf’ size 2M; 
```sql
alter database datafile '/u01/app/oracle/oradata/myinst/system01myinst.dbf' autoextend on next 100M;
CREATE TABLESPACE RONLY DATAFILE '/u01/app/oracle/oradata/myinst/Disk1/ronly01.dbf' SIZE 2M; 
```

4. Donner la liste des tablespaces par défaut
```sql
startup upgrade pfile=/u01/app/oracle/admin/myinst/pfile/initmyinst.ora
select * from database_properties where property_name like '%TABLESPACE';
```
>DEFAULT_TEMP_TABLESPACE  ->  TEMPTS1
Name of default temporary tablespace
DEFAULT_PERMANENT_TABLESPACE -> SYSTEM
Name of default permanent tablespace
5. Mettre le tablespace DATA01 comme tablespace par défaut. Vérifier votre opération.
```sql
ALTER DATABASE DEFAULT TABLESPACE  DATA01'; 
```
>Database altered.
6. Quels sont les tablespaces temporaires par défaut actuels?
```sql
select * from database_properties where property_name like '%TABLESPACE';
```
>DEFAULT_TEMP_TABLESPACE -> TEMPTS1
Name of default temporary tablespace
7. Afficher le liste des tablespaces avec la clause de stockage par défaut de chaque tablespace 
```sql
select  storage from database_properties;
```
8. Ajouter le tablespace TEMP comme tablespace temporaire par défaut 
```sql
ALTER DATABASE DEFAULT TEMPORARY TABLESPACE  TEMP;
```
>Database altered.

9. Quelles sont les deux manières pour augmenter la taille d’un tablespace? 

- Augmenter la taille d’un fichier existant 
```sql
alter database datafile '/u01/app/oracle/oradata/myinst/fichier.dbf' autoextend on next 100M;
```
- Ajouter un nouveau fichier 
```sql
ALTER TABLESPACE users ADD DATAFILE '/u01/app/oracle/oradata/myinst/fichier.dbf' size 100M;
```


10. Déplacer le tablespace INDX01 vers DISK1
```sh
cd /u01/app/oracle/oradata/myinst/Disk3/
mv indx01.dbf /u01/app/oracle/oradata/myinst/Disk1
```

11. Mettre le tablespace RONLY en lecture seule après y avoir créé une table. 
```sql
ALTER TABLESPACE RONLY READ ONLY;
```
>Tablespace altered.

12. Supprimez le tablespace RONLY et vérifiez l’opération. 
```sql
drop tablespace RONLY INCLUDING CONTENTS; 
```
>Tablespace dropped.
```sql
select tablespace_name from dba_tablespaces;
```
|TABLESPACE_NAME|
|--------|
|SYSTEM|
|SYSAUX|
|UNDOTBS1|
|TEMPTS1|
|DATA01|
|TEMP|
|INDX01|

13. Afficher la liste des tablespaces, les fichiers associés et la taille de chaque fichier 
```sql
SELECT tablespace_name, FILE_NAME, BLOCKS, FROM DBA_DATA_FILES;
```
|TABLESPACE_NAME|FILE_NAME|BLOCKS|
|--------|--------|--------|
| SYSTEM  |   /u01/app/oracle/oradata/myinst/system01myinst.d  |  25600  |
| SYSAUX  |   /u01/app/oracle/oradata/myinst/sysaux01.dbf  |  41600  |
| UNDOTBS1  |  /u01/app/oracle/oradata/myinst/undotbs01.dbf |  35968  |  
| DATA01  |   /u01/app/oracle/oradata/myinst/Disk4/data01.dbf |  256  | 
| INDX01  |  /u01/app/oracle/oradata/myinst/Disk3/indx01.dbf  |  384  | 


14. Afficher la taille totale et le nombre de fichier de chaque tablespace

15. Afficher la taille de l’espace libre dans chaque tablespace 
```sql
SELECT TABLESPACE_NAME , FILE_NAME MAX(blocks) "MAXIMUM", MIN(blocks) "MINIMUM", SUM(blocks) "TOTAL" FROM DBA_FREE_SPACE GROUP BY TABLESPACE_NAME,  FILE_NAME;
```

16. Expliquer ce que fait la requête suivante? Exécuter la requête et interpréter le résultat. 
```sql
SELECT   a.tablespace_name,   ROUND   (((c.BYTES - NVL   (b.BYTES,   0))   /   c.BYTES) * 100,2) percentage_used,  c.BYTES  /  1024  /  1024  space_allocated,  ROUND  (c.BYTES  / 1024  /  1024 - NVL (b.BYTES, 0) / 1024 / 1024,2) space_used, ROUND (NVL (b.BYTES, 0) / 1024 / 1024, 2) space_free,c.DATAFILES  FROM  dba_tablespaces  a,  (  SELECT  tablespace_name,  SUM  (BYTES)  BYTES  FROM dba_free_space GROUP BY tablespace_name ) b, (SELECT COUNT (1) DATAFILES, SUM (BYTES) BYTES, ablespace_name FROM dba_data_files GROUP BY tablespace_name ) c WHERE b.tablespace_name(+) = a.tablespace_name AND c.tablespace_name(+) = a.tablespace_name ORDER BY NVL (((c.BYTES - NVL (b.BYTES, 0)) / c.BYTES), 0) DESC;
```
> Cette requête permet de trouver de l'espace libre sur l'espace de table 


# TP4 - Gestion de la sécurité

### Gestion des utilisateurs

1. Créez  l’utilisateur  bob  avec  le  mot  de  passe  along  avec  comme  tablespace  par défaut DATA01. 
```sql
CREATE USER bob IDENTIFIED BY along DEFAULT TABLESPACE DATA01;
```
>User created.


2. Créez un utilisateur kay avec le mot de passe mary et donner lui les droits nécessaires pour pouvoir se connecter et créer des objets. 
```sql
CREATE USER kay IDENTIFIED BY mary;
```
>User created.
```sql
GRANT CONNECT,CREATE table TO kay;
```
>Grant succeeded.



a) Ouvrir une deuxième session sqlplus et se connecter en tant que kay.
```sh
sqlplus
jay
mary
```
b. A partir de la session kay, copier la table EMP à partir du schéma SCOTT dans le compte de kay. Que se passe-t-il? Corriger le problème et refaire une tentative de création de la table emp par kay

```sql
CREATE TABLE EMP AS ( SELECT * FROM SCOTT.EMP);
```
3. A partir du dictionnaire de données (vueDBA_Users), affichez les informations (username, l’état du compte, tablespace par défaut) sur bob et kay. 
```sql

```

5.  En  tant  que  SYSTEM,  alloué  à  kay  un  quota  de  5M  sur  le  tablespace  DATA01. Vérifier  votre opération
```sql
ALTER USER KAY QUOTA 5M ON DATA01;
```
>User altered.
6. En tant que SYSTEM, supprimez le quota de kay sur son tablespace par défaut.
```sql
ALTER USER KAY QUOTA 0 ON DATA01;
```
>User altered.
7. Supprimez le compte de kay drop user kay cascade; 
```sql
DROP USER kay CASCADE.
```

8. bob a oublié son mot de passe. Allouez-lui le mot de passe olink et demandez lui de changer son mot de passe à sa prochaine connexion
```sql
ALTER USER bob IDENTIFIED BY olink PASSWORD EXPIRE;
```
>User altered.


9. Lancez un shell et connectez-vous en tant que bob. Que se passe-t-il? 
>On lui demande de changer son mot de passe immediatement.

# Gestion des profils
1. Affichez les informations sur les profils, puis toutes les ressources du profil DEFAULT
```sql
SELECT USERNAME, PROFILE, ACCOUNT_STATUS FROM DBA_USERS;
```

|USERNAME|PROFILE|ACCOUNT_STATUS|
|--------|--------|--------|
| MGMT_VIEW  |  DEFAULT  |  OPEN  |
| SYS  |   DEFAULT  |  OPEN  |
| SYSTEM  | DEFAULT |  OPEN  | 

>37 rows selected.

2.(a)  Créez  un  nouveau  profil  de  telle  sorte  que  2  sessions  simultanées  par  utilisateur  soient permises et que le temps d’inactivité soit limité à une minute. Vérifier que votre opération s’est bien exécutée
```sql
CREATE PROFILE myprofil LIMIT SESSIONS_PER_USER  2 CONNECT_TIME 1 ; 
```
>Profile created.

b) Allouez ce profil à bob et vérifier votre opération
```sql
ALTER USER bob PROFILE myprofil; 
```
>User altered.

- Activer la limitation des ressources. 
```sql
ALTER SYSTEM SET RESOURCE_LIMIT = TRUE;
```
>System altered.

c. En utilisant SQLPLUS, tentez d’ouvrir 3 sessions pour l’utilisateur bob.

>ORA-02391: exceeded simultaneous SESSIONS_PER_USER limit

# Gestion des privilèges

1.En tant que SYSTEM, créez l’utilisateur kay et donnez-lui la possibilité de se connecter à la 
base de données et de créer des objets dans son schéma. 
```sql

```


2.(a)  Connectez-vous  en  tant  que  SCOTT/tiger  et  donnez  à  kay  le  privilège  select  sur les SCOTT.EMP et SCOTT.dep. 
```sql

```


(b)  Connectez-vous  en  tant  que  kay  et  créez  deux  tables  EMP  et  DEPT  en  copiant  les 
données des tables correspondantes de scott.

```sql

```


(c)  Connectez-vous  en  tant  que  SYSTEM/oracle,  accordez  à  bob  la  possibilité  de sélectionner des données dans la table DEPT de kay. 

```sql

```


(d) Donnez également à bob le privilège de faire un select sur la table EMP de kay avec le 
droit d’administrer ce privilège 

```sql

```

3.(a) Examinez les vues du dictionnaire de données qui enregistrent ces informations. 
```sql

```















