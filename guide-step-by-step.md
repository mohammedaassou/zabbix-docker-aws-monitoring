# Guide étape par étape — Infrastructure cloud de supervision centralisée sous AWS (Zabbix Docker)

Ce guide suit le cahier des charges du projet :
- 1 VPC + 1 subnet public (sans VPN)
- 3 EC2 :
  - Zabbix Server (Ubuntu) : `t3.large`
  - Client Linux (Ubuntu) : `t3.medium`
  - Client Windows Server : `t3.large`
- Zabbix déployé en conteneurs Docker (server + web + DB)
- Monitoring des 2 clients (statut ZBX vert, graph CPU/RAM, démo alerte)

## 0) Pré-requis
- Compte AWS Learner Lab (budget ~50$)
- Région : **us-east-1 (N. Virginia)**
- Une IP publique “stable” pour toi (ton réseau actuel) afin de limiter les accès (règle SG « My IP »).

## 1) Préparer le dépôt GitHub (obligatoire)
1. Crée un dépôt GitHub (public ou privé).
2. Pousse ce projet (ce workspace) :
   - `git add .`
   - `git commit -m "Initial project guide + compose"`
   - `git remote add origin <URL>`
   - `git push -u origin main`
3. Dans ton compte rendu, mets le lien du dépôt (intro ou fin de rapport).

## 2) Créer le VPC + Subnet public
Objectif : simplifier l’accès sans VPN (conforme au sujet).

1. AWS Console → **VPC** → Create VPC (VPC and more conseillé)
2. Paramètres recommandés :
   - 1 VPC (CIDR ex: `10.0.0.0/16`)
   - 1 subnet public (CIDR ex: `10.0.1.0/24`)
   - IGW (Internet Gateway) attachée
   - Route table : route `0.0.0.0/0` → IGW
3. Active “Auto-assign public IPv4” sur le subnet public.

Captures recommandées (rapport) :
- Figure 1 : Création du VPC
- Figure 2 : Subnet public + Route Table + IGW

## 3) Créer les Security Groups (SG)
But : autoriser **uniquement** les ports demandés.

### 3.1 SG Zabbix Server (EC2 Ubuntu)
Inbound (entrants) :
- HTTP `80` depuis **ton IP**
- HTTPS `443` depuis **ton IP** (optionnel si tu ne configures pas TLS)
- Zabbix server `10051` depuis le SG des clients (ou depuis le CIDR du subnet)
- SSH `22` depuis **ton IP**

Outbound : laisse “All traffic” (simple pour le lab).

### 3.2 SG Client Linux
Inbound :
- SSH `22` depuis **ton IP**
- Zabbix agent `10050` depuis **SG Zabbix Server** (recommandé)

### 3.3 SG Client Windows
Inbound :
- RDP `3389` depuis **ton IP**
- Zabbix agent `10050` depuis **SG Zabbix Server**

Captures recommandées :
- Figure 3 : SG Zabbix (règles inbound)
- Figure 4 : SG Clients (règles inbound)

## 4) Lancer les 3 instances EC2 (dans le subnet public)
AWS Console → **EC2** → Launch instance.

### 4.1 EC2 Zabbix (Ubuntu)
- Type : `t3.large`
- OS : Ubuntu LTS
- Network : VPC + subnet public
- Auto-assign public IP : enabled
- Security group : SG Zabbix Server
- Tag obligatoire (preuve) : ex `NomPrenom-Zabbix-Server`

### 4.2 EC2 Client Linux (Ubuntu)
- Type : `t3.medium`
- Security group : SG Client Linux
- Tag : `NomPrenom-Client-Linux`

### 4.3 EC2 Client Windows Server
- Type : `t3.large`
- OS : Windows Server
- Security group : SG Client Windows
- Tag : `NomPrenom-Client-Windows`

Captures recommandées :
- Figure 5 : Les 3 instances “Running” + tags visibles

## 5) Déployer Zabbix conteneurisé sur l’EC2 Ubuntu (serveur)
Objectif : installer Docker + Docker Compose, puis lancer `docker-compose.yml`.

### 5.1 Connexion SSH
Depuis ton terminal :
- `ssh -i <ta_cle.pem> ubuntu@<IP_PUBLIQUE_EC2_ZABBIX>`

### 5.2 Installer Docker + plugin Compose
Sur Ubuntu :
- `sudo apt-get update`
- `sudo apt-get install -y ca-certificates curl gnupg`
- `sudo install -m 0755 -d /etc/apt/keyrings`
- `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg`
- `echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`
- `sudo apt-get update`
- `sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`
- `sudo usermod -aG docker ubuntu`
- Déconnexion/reconnexion SSH (ou `newgrp docker`).

### 5.3 Déployer avec Docker Compose
1. Copie le fichier `docker-compose.yml` du dépôt sur l’EC2 (méthodes possibles : `scp`, Git clone, ou copier-coller).
2. Dans le dossier :
   - `docker compose up -d`
3. Vérification :
   - `docker ps`
   - `docker logs zabbix-server --tail 50`

### 5.4 Accès Web
Navigateur : `http://<IP_PUBLIQUE_EC2_ZABBIX>/`
- Login : `Admin`
- Password : `zabbix`

Capture obligatoire :
- Figure 6 : Connexion Zabbix réussie (interface visible)

## 6) Installer et configurer l’agent Zabbix sur le client Linux (Ubuntu)
Objectif : que Zabbix voie l’hôte en **vert** (ZBX).

### 6.1 Installer Zabbix Agent 2
Sur le client Linux (SSH) :
1. Télécharger et installer le repo Zabbix (adapte la version Ubuntu si besoin) :
   - `wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-2+ubuntu22.04_all.deb`
   - `sudo dpkg -i zabbix-release_7.0-2+ubuntu22.04_all.deb`
   - `sudo apt-get update`
2. Installer l’agent :
   - `sudo apt-get install -y zabbix-agent2`

### 6.2 Configurer `zabbix_agent2.conf`
Édite `/etc/zabbix/zabbix_agent2.conf` :
- `Server=<IP_PRIVEE_EC2_ZABBIX>`
- `ServerActive=<IP_PRIVEE_EC2_ZABBIX>`
- `Hostname=NomPrenom-Client-Linux` (doit matcher le nom dans l’UI)

Puis :
- `sudo systemctl enable --now zabbix-agent2`
- `sudo systemctl status zabbix-agent2 --no-pager`

Capture obligatoire :
- Figure 7 : extrait de la config (Server/ServerActive/Hostname)

## 7) Installer et configurer l’agent Zabbix sur le client Windows
### 7.1 Connexion RDP
- Récupère le mot de passe admin Windows (Decrypt password avec ta clé).
- RDP vers `IP_PUBLIQUE_EC2_WINDOWS`.

### 7.2 Installer l’agent
Option simple (recommandée) : MSI officiel Zabbix Agent 2.
- Télécharge Zabbix Agent 2 (Windows) depuis le site Zabbix.
- Lors de l’installation :
  - Zabbix server : `<IP_PRIVEE_EC2_ZABBIX>`
  - Hostname : `NomPrenom-Client-Windows`

Sinon : installe en silent + config manuelle (si tu préfères).

Capture obligatoire :
- Figure 8 : config Windows (Hostname + Server)

## 8) Ajouter les hôtes dans Zabbix et vérifier le statut (ZBX vert)
Dans l’UI Zabbix :

1. **Data collection → Hosts → Create host**
2. Hôte Linux :
   - Host name : `NomPrenom-Client-Linux`
   - Interface Agent : IP privée du client Linux, port `10050`
   - Template : `Linux by Zabbix agent`
3. Hôte Windows :
   - Host name : `NomPrenom-Client-Windows`
   - Interface Agent : IP privée du client Windows, port `10050`
   - Template : `Windows by Zabbix agent`

Vérification :
- **Data collection → Hosts** : colonne Availability doit afficher **ZBX vert**.

Captures obligatoires :
- Figure 9 : Statut ZBX vert pour Linux + Windows

## 9) Monitoring & tableaux de bord (graph CPU/RAM)
Objectif : montrer des données en temps réel.

1. **Monitoring → Latest data**
2. Filtre sur un host (Linux ou Windows)
3. Ouvre une métrique CPU/RAM et affiche le graphe.

Capture obligatoire :
- Figure 10 : Graphe CPU ou RAM d’un client

## 10) Démo alerte (pour la vidéo et/ou le rapport)
Option simple : simuler une panne d’agent.
- Sur le client Linux : `sudo systemctl stop zabbix-agent2`
- Attends 1–2 minutes, puis vérifie dans Zabbix (problème/availability).
- Redémarre : `sudo systemctl start zabbix-agent2`

Capture (optionnelle mais très valorisante) :
- Figure 11 : Alerte / problème détecté puis résolu

## 11) Conseils pour un rapport “excellent” (captures + preuves)
- Inclure ton identité : username AWS visible + tags `NomPrenom-*`.
- Numéroter chaque image + légende :
  - “Figure X : …”
- Qualité : screenshots (pas photo téléphone).
- Annotations : entourer en rouge IP publique, statut agent, etc.

## 12) Structure du compte rendu (PDF)
Obligatoire : page de garde avec
- Logo établissement
- Titre du projet
- Nom étudiant
- Encadrant : Prof. Azeddine KHIAT
- Année : 2025/2026
- Filière

Sommaire recommandé :
1. Introduction : projet & outils (AWS, Docker, Zabbix)
2. Architecture réseau : schéma VPC + SG
3. Architecture EC2 : les 3 instances (types + OS)
4. Déploiement Zabbix : Docker/Compose + login UI
5. Configuration Agents : Linux + Windows
6. Monitoring : hosts OK + graph CPU/RAM
7. Conclusion : difficultés & solutions

## 13) Préparer la vidéo (5–10 minutes)
- Intro : objectif (page de garde), éventuellement visage
- AWS : montrer 3 instances “Running”
- Zabbix :
  1. Login UI
  2. Configuration → Hosts (agents connectés)
  3. Monitoring → Latest data (données live)
  4. Provoquer une alerte (arrêt agent) + retour à la normale
- Conclusion : acquis

## 14) Limitations critiques du Learner Lab
- Instances : rester sur `t3.medium` / `t3.large`
- Région : **us-east-1**
- Arrêt auto : après redémarrage EC2, relancer `docker compose up -d`
- Budget : Stop les instances quand tu ne travailles pas

---

## Check-list finale (avant rendu)
- [ ] 3 EC2 running + tags visibles
- [ ] Zabbix UI accessible sur port 80
- [ ] Hôte Linux : ZBX vert + données CPU/RAM
- [ ] Hôte Windows : ZBX vert + données CPU/RAM
- [ ] Captures numérotées + légendes
- [ ] Lien GitHub dans le rapport
- [ ] Vidéo 5–10 min prête
