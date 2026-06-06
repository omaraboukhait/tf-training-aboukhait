# 📘 Lab 4 — Terraform Core Behaviour & Remote State

## 🎯 Objectifs du Lab

À la fin de ce lab, vous aurez :

- Créé votre **bucket S3 personnel** pour stocker votre state Terraform
- Configuré un **backend distant S3** avec locking natif (`use_lockfile = true`)
- Compris le comportement de Terraform face à des **modifications hors Terraform**
- Compris le comportement face à des **modifications dans le code Terraform**

---

# 🧩 Partie A — Bootstrap : Créer votre bucket S3

> ⚠️ Cette partie se fait **une seule fois** — le bucket sera réutilisé pour tous les labs suivants (Lab 4 à Lab 14).

## Étape A1 — Préparer le dossier bootstrap

```bash
cd labs/lab4-state
mkdir bootstrap && cd bootstrap
```

## Étape A2 — Créer le bucket S3

```bash
# Remplacez <votre-prenom> par votre prénom en minuscules sans accent
USERNAME=<votre-prenom>
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="tf-training-${USERNAME}-${ACCOUNT_ID}"

echo "Bucket : $BUCKET_NAME"
```

```bash
# Créer le bucket
aws s3api create-bucket \
  --bucket $BUCKET_NAME \
  --region eu-west-1 \
  --create-bucket-configuration LocationConstraint=eu-west-1
```

```bash
# Activer le versioning (permet de récupérer un state précédent)
aws s3api put-bucket-versioning \
  --bucket $BUCKET_NAME \
  --versioning-configuration Status=Enabled
```

```bash
# Bloquer l'accès public
aws s3api put-public-access-block \
  --bucket $BUCKET_NAME \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

```bash
# Vérifier que le bucket est créé
aws s3 ls | grep $BUCKET_NAME
```

> 💡 Le nom du bucket suit le format `tf-training-<prenom>-<account-id>` — il est **unique** par participant et par compte AWS.

## Étape A3 — Sauvegarder le nom du bucket

```bash
# Notez votre bucket name — vous en aurez besoin dans tous les labs suivants
echo "Mon bucket S3 : $BUCKET_NAME" >> $HOME/tf-training-info.txt
cat $HOME/tf-training-info.txt
```

---

# 🧩 Partie B — Configurer le Backend S3

## Étape B1 — Retourner dans le dossier du lab

```bash
cd ..
# Vous êtes dans labs/lab4-state
```

## Étape B2 — Créer les fichiers

```
lab4-state/
├── bootstrap/          ← déjà fait
├── versions.tf
├── variables.tf
├── main.tf
└── outputs.tf
```

### `versions.tf`

```hcl
terraform {
  required_version = ">= 1.15.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    region       = "eu-west-1"
    use_lockfile = true
    encrypt      = true
  }
}

provider "aws" {
  region = "eu-west-1"
}
```

> 💡 Le bloc `backend "s3"` est volontairement **incomplet** — le `bucket` et la `key` seront passés au `terraform init` via `-backend-config` pour éviter de hardcoder le nom du bucket dans le code.

> 💡 **`use_lockfile = true`** — disponible depuis Terraform 1.11. Permet le locking du state directement sur S3 **sans DynamoDB**. Un fichier `.tflock` est créé dans le bucket pendant les opérations.

### `variables.tf`

```hcl
variable "username" {
  description = "Votre prenom - permet de differencier les ressources entre participants"
  type        = string
}
```

### `main.tf`

```hcl
resource "aws_security_group" "lab4_sg" {
  name        = "lab4-sg-${var.username}"
  description = "Security group Lab 4 - ${var.username}"

  tags = {
    Name     = "lab4-sg-${var.username}"
    Lab      = "lab4"
    Username = var.username
  }
}

resource "aws_instance" "lab4_instance" {
  ami                    = "ami-0694d931cee176e7d"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.lab4_sg.id]

  tags = {
    Name     = "lab4-instance-${var.username}"
    Lab      = "lab4"
    Username = var.username
  }
}
```

### `outputs.tf`

```hcl
output "instance_id" {
  description = "ID de l'instance EC2"
  value       = aws_instance.lab4_instance.id
}

output "security_group_id" {
  description = "ID du Security Group"
  value       = aws_security_group.lab4_sg.id
}
```

---

# 🧩 Partie C — Initialiser avec le Backend S3

## Étape C1 — terraform init avec backend-config

```bash
# Remplacez <votre-prenom> et <account-id> par vos valeurs
terraform init \
  -backend-config="bucket=tf-training-<votre-prenom>-<account-id>" \
  -backend-config="key=lab4/terraform.tfstate"
```

Résultat attendu :
```
Initializing the backend...
Successfully configured the backend "s3"!
```

> 💡 Le state sera stocké dans S3 à l'adresse :
> `s3://tf-training-<prenom>-<account-id>/lab4/terraform.tfstate`

## Étape C2 — Vérifier la configuration du backend

```bash
terraform init -reconfigure \
  -backend-config="bucket=tf-training-<votre-prenom>-<account-id>" \
  -backend-config="key=lab4/terraform.tfstate"
```

## Étape C3 — Déployer

```bash
terraform apply -var="username=<votre-prenom>"
```

Tapez `yes` pour confirmer.

## Étape C4 — Vérifier que le state est dans S3

```bash
aws s3 ls s3://tf-training-<votre-prenom>-<account-id>/lab4/
```

Résultat attendu :
```
terraform.tfstate
```

---

# 🧩 Partie D — Modifier une ressource hors Terraform

## Étape D1 — Modifier un tag depuis la console AWS

```bash
INSTANCE_ID=$(terraform output -raw instance_id)

# Ajouter un tag manuellement (hors Terraform)
aws ec2 create-tags \
  --resources $INSTANCE_ID \
  --tags Key=ModifiedOutside,Value=true
```

## Étape D2 — Observer le comportement de Terraform

```bash
terraform plan -var="username=<votre-prenom>"
```

> 📋 Terraform détecte que la ressource a été modifiée hors de son contrôle et propose de **remettre** la configuration au desired state.

## Étape D3 — Refresh du state

```bash
terraform refresh -var="username=<votre-prenom>"
```

```bash
# Vérifier que le state a capturé le tag
terraform state show aws_instance.lab4_instance | grep ModifiedOutside
```

---

# 🧩 Partie E — Modifier dans le code Terraform

## Étape E1 — Changer le type d'instance dans main.tf

Dans `main.tf`, modifiez :

```hcl
resource "aws_instance" "lab4_instance" {
  ...
  instance_type = "t2.small"   # ← changer t2.micro en t2.small
  ...
}
```

## Étape E2 — Observer le plan

```bash
terraform plan -var="username=<votre-prenom>"
```

> 📋 Terraform affiche qu'il va **modifier** l'instance (`~`) pour changer le type. C'est la différence entre **desired state** (code) et **current state** (AWS).

## Étape E3 — Annuler la modification

Remettez `t2.micro` dans `main.tf` avant de continuer.

```bash
terraform plan -var="username=<votre-prenom>"
```

Résultat attendu : `No changes. Infrastructure is up-to-date.`

---

# 🧩 Partie F — Tester le locking S3

Ouvrez **deux terminaux** simultanément dans le même dossier.

- **Terminal 1** : `terraform apply -var="username=<votre-prenom>"`
- **Terminal 2** : `terraform apply -var="username=<votre-prenom>"` immédiatement après

Le Terminal 2 affiche :
```
Error: Error acquiring the state lock
```

> 💡 Le fichier `.tflock` créé par `use_lockfile = true` empêche deux opérations simultanées sur le même state.

Vérifiez le fichier de lock dans S3 :

```bash
aws s3 ls s3://tf-training-<votre-prenom>-<account-id>/lab4/
```

---

# 🧩 Partie G — Nettoyage

```bash
terraform destroy -var="username=<votre-prenom>"
```

> ⚠️ Ne supprimez **pas** votre bucket S3 — il sera réutilisé dans tous les labs suivants !

---

# ✅ Validation du Lab

| # | Critère | Validé |
|---|---------|--------|
| 1 | Bucket S3 `tf-training-<prenom>-<account-id>` créé | ☐ |
| 2 | `terraform init -backend-config` réussi | ☐ |
| 3 | State visible dans S3 après `terraform apply` | ☐ |
| 4 | `terraform plan` détecte la modification hors Terraform | ☐ |
| 5 | `terraform refresh` capture le tag ajouté manuellement | ☐ |
| 6 | `terraform plan` détecte le changement `t2.small` | ☐ |
| 7 | Locking S3 bloque le 2ème `apply` simultané | ☐ |
| 8 | `terraform destroy` supprime les ressources AWS | ☐ |
| 9 | Bucket S3 conservé pour les labs suivants | ☐ |

---

# 🛠️ Résolution des problèmes courants

| Problème | Cause | Solution |
|----------|-------|----------|
| `BucketAlreadyExists` | Nom de bucket déjà pris | Vérifier que le nom inclut bien votre prénom et account-id |
| `Backend init error` | Bucket ou key incorrects | Vérifier les valeurs `-backend-config` |
| `Error acquiring state lock` | Lock non libéré | `terraform force-unlock <ID>` |
| `NoSuchBucket` | Bucket non créé | Relancer les commandes de bootstrap |
| `use_lockfile not supported` | Version Terraform < 1.11 | Vérifier `terraform version` |

---

🎉 **Fin du Lab 4 — Votre remote state S3 est configuré pour toute la formation !**

> Le Lab 5 introduit les **outputs** et les **variables** pour rendre votre code dynamique.
