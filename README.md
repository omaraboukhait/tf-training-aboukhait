# 🟣 Terraform Training — Repo Template

Bienvenue dans le repo de formation **Terraform + AWS**.  
Ce repo sert de base pour chaque participant : vous allez le forker, l'alimenter lab après lab, et repartir avec un projet Terraform fonctionnel à la fin de la formation.

---

## 🛠️ Environnement

| Outil | Version |
|-------|---------|
| Terraform | `1.15.5` |
| AWS CLI | `v2 (latest)` |
| OS (Codespace) | Ubuntu 24.04 |

L'environnement est **entièrement préconfiguré** via GitHub Codespaces — aucune installation locale nécessaire.

---

## 📁 Structure du repo

```
tf-training-<prenom>/
├── .devcontainer/
│   ├── devcontainer.json     ← Config Codespace (extensions VSCode)
│   └── Dockerfile            ← Image (Terraform 1.15.5 + AWS CLI v2)
│
├── labs/
│   ├── lab1-setup/           ← README uniquement (pas de code à écrire)
│   ├── lab2-basics/          ← Premier fichier Terraform
│   ├── lab3-variables/       ← Variables, outputs, locals
│   ├── lab4-modules/         ← Modules réutilisables
│   └── lab5-remote-state/    ← State distant sur S3
│
├── solutions/                ← Correction des labs (débloquée par le formateur)
│   ├── lab2/
│   ├── lab3/
│   ├── lab4/
│   └── lab5/
│
└── README.md                 ← Ce fichier
```

---

## 🧪 Labs

| # | Lab | Thème |
|---|-----|-------|
| 1 | [Lab 1 — Setup](./labs/lab1-setup/README.md) | Codespace + accès AWS |
| 2 | [Lab 2 — Basics](./labs/lab2-basics/README.md) | Provider, Resource, Plan, Apply |
| 3 | [Lab 3 — Variables](./labs/lab3-variables/README.md) | Variables, Outputs, Locals |
| 4 | [Lab 4 — Modules](./labs/lab4-modules/README.md) | Modules réutilisables |
| 5 | [Lab 5 — Remote State](./labs/lab5-remote-state/README.md) | Backend S3 + DynamoDB |

---

## 🚀 Démarrer

1. Cliquer sur **Use this template** → **Create a new repository**
2. Nommer le repo `terraform-training-<votre-prenom>`
3. Ouvrir un **Codespace** sur le repo (`Code` → `Codespaces` → `Create codespace on main`)
4. Suivre les instructions du **Lab 1**

---

## 📌 Conventions

- Tout le code Terraform se place dans le dossier du lab correspondant
- On ne modifie **jamais** le dossier `solutions/` pendant les labs
- Les commandes `terraform` se lancent **depuis le dossier du lab** :

```bash
cd labs/lab2-basics
terraform init
terraform plan
terraform apply
```

---

*Formation animée par **[Nom du formateur]** — Tous droits réservés.*
