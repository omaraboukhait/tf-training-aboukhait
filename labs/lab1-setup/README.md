# 📘 Lab 1 — Préparation de l'Espace de Travail

## 🎯 Objectifs du Lab

À la fin de ce lab, chaque participant aura :

- Un **repo GitHub personnel** basé sur le template de formation
- Un **Codespace GitHub** opérationnel avec Terraform et AWS CLI pré-installés
- Ses **credentials AWS** configurés et vérifiés dans le Codespace
- Un environnement prêt pour tous les Labs suivants

---

# 🧩 Étape 0 — Prérequis

Avant de commencer, assurez-vous d'avoir :

- Un **compte GitHub** actif (gratuit) → https://github.com/signup
- Un **navigateur moderne** (Chrome, Firefox, Edge)
- Les **informations d'accès AWS** fournies par le formateur :
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
  - `AWS_DEFAULT_REGION` (ex : `eu-west-1`)

> ℹ️ Si vous n'avez pas encore de compte GitHub, créez-en un maintenant avant de continuer.

---

# 🧩 Étape 1 — Créer votre repo depuis le template

1. Ouvrez le lien du **repo template** fourni par le formateur :

   👉 **https://github.com/SAIDI-HAMZA/tf-training-template**

2. En haut à droite, cliquez sur **Use this template** → **Create a new repository**

3. Renseignez les informations suivantes :
   - **Repository name** : `terraform-training-<votre-prenom>`  
     *(ex : `terraform-training-alice`)*
   - **Owner** : votre compte GitHub personnel
   - **Visibility** : `Private` (recommandé)

4. Cliquez sur **Create repository from template**

✅ Vous disposez maintenant de votre propre copie du repo de formation.

---

# 🧩 Étape 2 — Démarrer votre GitHub Codespace

1. Sur la page de **votre** repo, cliquez sur le bouton vert **`< > Code`**
2. Sélectionnez l'onglet **Codespaces**
3. Cliquez sur **Create codespace on main**

> ⏳ La création du Codespace prend **1 à 3 minutes** lors du premier démarrage — c'est normal, le conteneur se construit.

Une fois prêt, VS Code s'ouvre dans votre navigateur avec :
- ✅ **Terraform** pré-installé
- ✅ **AWS CLI v2** pré-installé
- ✅ **Extension HashiCorp Terraform** pour VS Code
- ✅ **jq** disponible (traitement JSON)

> 💡 **Toutes les commandes des labs suivants se font dans le Terminal intégré du Codespace.**  
> Pour ouvrir le terminal : menu **Terminal** → **New Terminal**, ou raccourci `` Ctrl+` ``

---

# 🧩 Étape 3 — Vérifier les outils installés

Dans le terminal du Codespace, exécutez les commandes suivantes :

```bash
# Vérifier Terraform
terraform version
```

Résultat attendu :
```
Terraform v1.15.5
on linux_amd64
```

```bash
# Vérifier AWS CLI
aws --version
```

Résultat attendu :
```
aws-cli/2.x.x Python/3.x.x Linux/...
```

```bash
# Vérifier jq
jq --version
```

Résultat attendu :
```
jq-1.6
```

> ❌ Si une commande n'est pas reconnue, signalez-le au formateur.

---

# 🧩 Étape 4 — Configurer l'accès AWS

Vous allez configurer vos credentials AWS dans le Codespace. Il existe deux méthodes, selon les accès fournis par le formateur.

---

## Méthode A — Variables d'environnement (recommandée pour les labs)

Dans le terminal, exportez vos credentials directement :

```bash
export AWS_ACCESS_KEY_ID="AKIAXXXXXXXXXXXXXXXXX"
export AWS_SECRET_ACCESS_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
export AWS_DEFAULT_REGION="eu-west-1"
```

> ⚠️ Remplacez les valeurs par celles fournies par le formateur.  
> Ces variables sont **temporaires** : elles disparaissent si vous fermez le terminal.

Pour les rendre **persistantes dans votre session Codespace** :

```bash
echo 'export AWS_ACCESS_KEY_ID="AKIAXXXXXXXXXXXXXXXXX"' >> ~/.bashrc
echo 'export AWS_SECRET_ACCESS_KEY="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"' >> ~/.bashrc
echo 'export AWS_DEFAULT_REGION="eu-west-1"' >> ~/.bashrc
source ~/.bashrc
```

---

## Méthode B — Profil AWS CLI (alternative)

```bash
aws configure
```

Renseignez les informations demandées :
```
AWS Access Key ID [None]: AKIAXXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Default region name [None]: eu-west-1
Default output format [None]: json
```

Les credentials sont stockés dans `~/.aws/credentials`.

---

# 🧩 Étape 5 — Vérifier la connexion AWS

Exécutez la commande suivante pour valider que vos credentials fonctionnent :

```bash
aws sts get-caller-identity
```

Résultat attendu (exemple) :
```json
{
    "UserId": "AIDAXXXXXXXXXXXXXXXXX",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/terraform-training-alice"
}
```

> ✅ Si vous obtenez un JSON avec `Account` et `Arn`, votre accès AWS est opérationnel.  
> ❌ Si vous obtenez une erreur `InvalidClientTokenId` ou `AuthFailure`, vérifiez vos credentials avec le formateur.

---

Vérifiez aussi que vous avez bien les droits pour lister les régions :

```bash
aws ec2 describe-regions --output table
```

Vous devriez voir une liste de régions AWS s'afficher.

---

# 🧩 Étape 6 — Explorer la structure du repo

Le repo de formation contient l'arborescence suivante :

```
terraform-training-<prenom>/
├── .devcontainer/
│   ├── devcontainer.json     ← Configuration du Codespace
│   └── Dockerfile            ← Image utilisée (Terraform + AWS CLI)
├── labs/
│   ├── lab1-setup/
│   ├── lab2-basics/
│   └── ...
└── README.md
```

Explorez le fichier `.devcontainer/devcontainer.json` pour comprendre comment le Codespace est configuré :

```bash
cat .devcontainer/devcontainer.json
```

Vous pouvez voir les **extensions VS Code** installées automatiquement, notamment `HashiCorp.terraform`.

---

# ✅ Validation du Lab

Cochez chaque point avant de passer au Lab 2 :

| # | Critère | Validé |
|---|---------|--------|
| 1 | Repo GitHub créé depuis le template | ☐ |
| 2 | Codespace démarré et VS Code ouvert | ☐ |
| 3 | `terraform version` retourne `v1.14.0` | ☐ |
| 4 | `aws --version` retourne une version v2.x | ☐ |
| 5 | `aws sts get-caller-identity` retourne votre ARN | ☐ |
| 6 | `aws ec2 describe-regions` liste les régions | ☐ |

---

# 🛠️ Résolution des problèmes courants

| Problème | Cause probable | Solution |
|----------|---------------|----------|
| Le Codespace reste bloqué à "Building" | Première création longue | Patienter 3–5 min, actualiser la page |
| `terraform: command not found` | Conteneur non terminé | Fermer/rouvrir le terminal, ou `rebuild container` |
| `InvalidClientTokenId` | Mauvais Access Key ID | Vérifier la valeur copiée (pas d'espace) |
| `AuthFailure` | Mauvais Secret Access Key | Re-copier le secret depuis le formateur |
| `ExpiredTokenException` | Token temporaire expiré | Demander de nouveaux credentials |
| Codespace ne se lance pas | Limite GitHub gratuite atteinte | Vérifier les quotas sur github.com/codespaces |

---

🎉 **Fin du Lab 1 — Votre environnement de travail est prêt !**

> Le Lab 2 commence ici : vous allez écrire votre premier fichier Terraform avec le bloc `provider` et déployer une ressource AWS.
