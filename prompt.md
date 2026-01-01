Je veux que tu rédiges un README très détaillé et structuré, rédigé comme un rapport professionnel, basé sur les consignes exactes suivantes du professeur :

===== CONTENU DU PROJET =====
Projet : Mise en œuvre d’une infrastructure cloud de supervision centralisée sous AWS.  
Objectif : Déployer Zabbix conteneurisé (Docker) dans AWS afin de monitorer un parc hybride (Linux & Windows).  
Architecture :  
- 1 VPC  
- 1 subnet public  
- Security Groups autorisant : 80, 443, 10050, 10051, 22, 3389  
- Instances EC2 :  
  1. Zabbix server (t3.large, Ubuntu)  
  2. Client Linux (t3.medium, Ubuntu)  
  3. Client Windows (t3.large, Windows Server)  

Le README doit respecter le cahier des charges, les contraintes du learner lab, et inclure les bonnes pratiques.

===== STRUCTURE OBLIGATOIRE DU DOCUMENT =====
Le README doit suivre strictement le plan ci-dessous.  
Il doit être écrit comme un rapport PDF (ton formel, pédagogique, clair).

1. Page de garde  
   - Titre du projet  
   - Logo établissement (mettre un placeholder)  
   - Nom étudiant  
   - Encadrant : Prof. Azeddine KHIAT  
   - Filière  
   - Année universitaire : 2025/2026  
   - Image du projet (depuis /images)

2. Introduction  
   - Contexte  
   - Objectifs techniques  
   - Technologies utilisées (AWS, Docker, Zabbix, Ubuntu, Windows Server)

3. Architecture Réseau  
   - Description du VPC  
   - Schéma réseau (insérer /images/vpc.png)  
   - CIDR, subnet public, IGW, route table  
   - Security Groups (ports autorisés)  
   - Figures numérotées comme demandé (ex : Figure 1 : Création du VPC)

4. Instances EC2  
   - Liste des instances + caractéristiques t3.medium, t3.large  
   - Raison du choix de chaque type  
   - OS utilisés  
   - Captures 

5. Déploiement du serveur Zabbix  
   - Installation Docker  
   - Installation Docker Compose  
   - docker-compose.yml détaillé  
   - Démarrage des conteneurs  
   - Vérification de l’interface Web 
   - Figures numérotées et légendées

6. Configuration des agents (Linux + Windows)  
   - Installation agent Zabbix Linux (apt install, service…)  
   - Fichier zabbix_agentd.conf (inclure extrait lisible)  
   - Installation agent Zabbix Windows (MSI installer)  
   - Configuration du Hostname et Server  
   - Captures depuis 

7. Monitoring et Tableaux de Bord  
   - Ajout des hôtes dans Zabbix  
   - Statut ZBX en vert  
   - Graphiques CPU/RAM  
   - Screenshots depuis
   - Figures numérotées

8. Difficultés rencontrées et solutions  
   - Ports bloqués  
   - Docker non démarré après stop du lab  
   - Limitations learner lab (us-east-1, stop instances, budget 50$)  
   - Solutions apportées

9. Conclusion  
   - Résumé des acquis  
   - Importance d’un monitoring hybride  
   - Améliorations futures possibles

10. Lien GitHub  
   - Lien vers le repo contenant :  
     • README  
     • Fichiers docker  
     • Scripts agents  
     • Captures d’écran  
     • Diagrammes  

===== EXIGENCES DE FORME =====
- Utiliser un Markdown propre pour GitHub.  
- Insérer les captures du dossier /images avec des légendes et des numéros.  
- Style professionnel, clair, détaillé, comme un rapport académique.  
- Ajouter tableaux, explications techniques, ports, schémas ASCII si utile.  
- Aucune phrase vide, pas de texte superficiel.  
- Le README doit être long, riche et complet.

Rédige maintenant le README final selon toutes les instructions ci-dessus.
