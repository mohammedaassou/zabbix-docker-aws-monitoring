# Rapport LaTeX (Overleaf / TeXShop)

Ce dossier contient le rapport académique complet au format LaTeX.

## Compilation

### Overleaf
1. Créer un nouveau projet Overleaf.
2. Importer le dossier `report/` et le dossier `images/` à la racine du projet Overleaf.
3. Compiler `report/main.tex`.

### TeXShop (macOS)
Depuis le terminal, à la racine du projet :

```bash
cd report
pdflatex main.tex
pdflatex main.tex
```

> Astuce : une double compilation permet d’actualiser la table des matières et la liste des figures.

## Images
Le rapport référence des images dans `../images/` :
- `../images/logo.png`
- `../images/project-cover.png`
- `../images/vpc.png`
- `../images/ec2-1.png`
- `../images/zabbix-login.png`
- `../images/agent-linux.png`
- `../images/agent-win.png`
- `../images/dashboard1.png`
- `../images/dashboard2.png`
