# 📘 Lab 1 — Préparation de l'Espace de Travail (AWS CloudShell)

## 🎯 Objectifs du Lab

À la fin de ce lab, chaque participant aura :

- Accès à **AWS CloudShell** avec Terraform et AWS CLI disponibles
- L'environnement de travail configuré et vérifié
- Le code de formation récupéré et prêt à utiliser
- Un environnement prêt pour tous les Labs suivants

---

# 🧩 Étape 0 — Prérequis

Avant de commencer, assurez-vous d'avoir :

- Un **navigateur moderne** (Chrome, Firefox, Edge)
- Les **informations d'accès à la console AWS** fournies par le formateur :
  - URL de la console AWS
  - `Account ID`
  - `Username`
  - `Password`

> ℹ️ Aucune installation n'est nécessaire sur votre poste — tout se passe dans le navigateur.

---

# 🧩 Étape 1 — Se connecter à la Console AWS

1. Ouvrez le lien fourni par le formateur :
   👉 **https://console.aws.amazon.com**

2. Sélectionnez **"IAM user"**

3. Renseignez :
   - **Account ID** : fourni par le formateur
   - **Username** : fourni par le formateur
   - **Password** : fourni par le formateur

4. Cliquez sur **Sign in**

✅ Vous êtes connecté à la console AWS.

---

# 🧩 Étape 2 — Ouvrir AWS CloudShell

1. En haut à droite de la console AWS, cliquez sur l'icône **CloudShell** (terminal `>_`)

2. Une fenêtre s'ouvre en bas de l'écran — attendez quelques secondes que l'environnement démarre

3. Vous pouvez aussi agrandir CloudShell en plein écran via l'icône **"Expand"** en haut à droite du terminal

> 💡 CloudShell est un terminal Linux directement dans votre navigateur. Vos credentials AWS sont **automatiquement configurés** — pas besoin de saisir de clés d'accès.

> ⚠️ CloudShell est disponible dans certaines régions uniquement. Si l'icône n'apparaît pas, vérifiez que vous êtes sur la région **eu-west-1 (Ireland)** en haut à droite de la console.

---

# 🧩 Étape 3 — Vérifier les outils disponibles

Dans le terminal CloudShell, exécutez les commandes suivantes :

```bash
# Vérifier AWS CLI (pré-installé et configuré automatiquement)
aws --version
```

Résultat attendu :
```
aws-cli/2.x.x Python/3.x.x Linux/...
```

```bash
# Vérifier que les credentials fonctionnent
aws sts get-caller-identity
```

Résultat attendu :
```json
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/terraform-training-alice"
}
```

```bash
# Vérifier jq (pré-installé)
jq --version
```

---

# 🧩 Étape 4 — Installer Terraform

Terraform n'est pas pré-installé dans CloudShell, mais l'installation est simple :

```bash
# Télécharger Terraform 1.15.5
wget -q -O /tmp/terraform.zip \
  https://releases.hashicorp.com/terraform/1.15.5/terraform_1.15.5_linux_amd64.zip

# Décompresser dans le dossier home (pas besoin de sudo)
unzip /tmp/terraform.zip -d $HOME/.local/bin/
rm /tmp/terraform.zip

# Vérifier l'installation
terraform version
```

Résultat attendu :
```
Terraform v1.15.5
on linux_amd64
```

> ⚠️ CloudShell redémarre après 20 minutes d'inactivité et repart sur une session fraîche. Dans ce cas, relancez l'installation de Terraform (étape 4). Vos **fichiers** dans `$HOME` sont conservés, mais les **binaires installés** sont perdus.

Pour éviter de répéter l'installation à chaque session, ajoutez un script de bootstrap :

```bash
# Créer un script de réinstallation rapide
cat > $HOME/install-terraform.sh << 'EOF'
#!/bin/bash
wget -q -O /tmp/terraform.zip \
  https://releases.hashicorp.com/terraform/1.15.5/terraform_1.15.5_linux_amd64.zip
unzip -o /tmp/terraform.zip -d $HOME/.local/bin/
rm /tmp/terraform.zip
echo "✅ Terraform $(terraform version --json | jq -r .terraform_version) installé"
EOF
chmod +x $HOME/install-terraform.sh

# La prochaine fois, une seule commande suffit :
# bash $HOME/install-terraform.sh
```

---

# 🧩 Étape 5 — Configurer l'environnement de travail

```bash
# Créer le dossier de formation
mkdir -p $HOME/terraform-training
cd $HOME/terraform-training

# Créer les dossiers des labs
mkdir -p lab1-setup lab2-basics lab3-state lab4-modules lab5-remote-state

# Vérifier la structure
ls -la
```

Résultat attendu :
```
drwxr-xr-x  lab1-setup
drwxr-xr-x  lab2-basics
drwxr-xr-x  lab3-state
drwxr-xr-x  lab4-modules
drwxr-xr-x  lab5-remote-state
```

> 💡 Ce dossier `$HOME/terraform-training` **persiste entre les sessions CloudShell** (stockage 1 Go). Vous retrouverez votre travail même si CloudShell redémarre.

---

# 🧩 Étape 6 — Vérification complète

```bash
# Vérifier Terraform
terraform version

# Vérifier AWS CLI et les credentials
aws sts get-caller-identity

# Vérifier les droits EC2
aws ec2 describe-regions \
  --query "Regions[?RegionName=='eu-west-1'].RegionName" \
  --output text

# Afficher la structure de travail
ls $HOME/terraform-training/
```

---

# ✅ Validation du Lab

| # | Critère | Validé |
|---|---------|--------|
| 1 | Connexion à la console AWS réussie | ☐ |
| 2 | CloudShell ouvert et opérationnel | ☐ |
| 3 | `aws sts get-caller-identity` retourne votre ARN | ☐ |
| 4 | `terraform version` retourne `v1.15.5` | ☐ |
| 5 | Script `install-terraform.sh` créé dans `$HOME` | ☐ |
| 6 | Dossier `terraform-training/` créé avec les sous-dossiers | ☐ |

---

# 🛠️ Résolution des problèmes courants

| Problème | Cause probable | Solution |
|----------|---------------|----------|
| Icône CloudShell absente | Mauvaise région | Changer la région en **eu-west-1** en haut à droite |
| `terraform: command not found` | Session CloudShell redémarrée | Relancer `bash $HOME/install-terraform.sh` |
| `unzip: command not found` | Rare sur CloudShell | `sudo yum install -y unzip` |
| `aws sts` retourne une erreur | Session AWS expirée | Se reconnecter à la console AWS |
| CloudShell bloqué au démarrage | Problème réseau | Actualiser la page, réessayer |

---

🎉 **Fin du Lab 1 — Votre environnement CloudShell est prêt !**

> Le Lab 2 commence ici : vous allez écrire votre premier fichier Terraform avec le bloc `provider` et déployer une ressource AWS.
