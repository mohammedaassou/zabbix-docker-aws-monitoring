## Schéma d’architecture — Supervision centralisée AWS (Zabbix Docker)

```mermaid
graph TD
    VPC["VPC (10.0.0.0/16)"]
    Subnet["Subnet public (10.0.1.0/24)"]
    IGW["Internet Gateway"]
    ZabbixServer["EC2 Zabbix Server (Ubuntu, t3.large)\nDocker: Zabbix Server + Web + DB"]
    ClientLinux["EC2 Client Linux (Ubuntu, t3.medium)\nZabbix Agent 2"]
    ClientWindows["EC2 Client Windows (Win Server, t3.large)\nZabbix Agent 2"]
    User["Utilisateur (My IP)\nSSH/RDP/Web"]

    VPC --> Subnet
    Subnet --> ZabbixServer
    Subnet --> ClientLinux
    Subnet --> ClientWindows
    Subnet --> IGW
    User -- SSH 22/Web 80/10051 --> ZabbixServer
    User -- SSH 22 --> ClientLinux
    User -- RDP 3389 --> ClientWindows
    ZabbixServer -- 10050 --> ClientLinux
    ZabbixServer -- 10050 --> ClientWindows
```

**Légende :**
- **VPC** : réseau privé AWS
- **Subnet public** : accès Internet via IGW
- **EC2 Zabbix Server** : Ubuntu, Docker, Zabbix Server/Web/DB
- **EC2 Client Linux** : Ubuntu, Zabbix Agent 2
- **EC2 Client Windows** : Windows Server, Zabbix Agent 2
- **User** : accès sécurisé (My IP) via SSH/RDP/Web
- **Ports ouverts** : 22 (SSH), 80 (HTTP), 3389 (RDP), 10050/10051 (Zabbix)

---

Pour une version graphique, tu peux t’inspirer de ce schéma et le dessiner dans draw.io ou un autre outil, puis l’ajouter dans le dossier `images/` et l’inclure dans le README :

```markdown
![Schéma d’architecture](images/schema-architecture.png)
```