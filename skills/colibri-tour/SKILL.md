---
name: colibri-tour
description: Installer et démarrer Colibri sur le Windows de l'utilisateur. Choisir Qwen3-Coder-30B sur 16 Go de RAM, Qwen3.8-Flash-Next seulement si la machine a au moins 24 Go. Demander avant tout téléchargement.
---

# Installer Colibri sur ce Windows

Cette page est écrite pour l'assistant à qui l'on a demandé de l'exécuter. Une personne peut la suivre aussi. Les commandes viennent de [docs/AI_SETUP.md](https://github.com/JustVugg/colibri/blob/main/docs/AI_SETUP.md). En cas de contradiction sur une commande, le fichier de Colibri gagne. La règle de choix du modèle, ici, gagne.

## Règles

- Demander avant de télécharger un modèle (19 Go et plus) et avant `winget` ou tout autre paquet système.
- Ne pas inventer de clé d'API et ne pas en écrire dans un fichier. Pour un usage local, ne pas ouvrir le port.
- Un seul modèle servi à la fois.
- Si une commande s'arrête, la relancer : le téléchargement reprend.

## 1. Colibri

Dans PowerShell, dossier de travail de l'utilisateur :

```powershell
py -3 --version
git clone https://github.com/JustVugg/colibri
cd colibri
```

Si `py` manque, montrer `winget install -e --id Python.Python.3.12 --scope user` et ne le lancer qu'après un oui. Si `git` manque, le dire et s'arrêter.

## 2. Ce que la machine peut tenir

```powershell
py -3 c\setup_hw.py --json
py -3 c\coli setup --list --json
```

Lire `memory.total`. Dire à l'utilisateur le modèle retenu, sa taille, et pourquoi les autres sont écartés.

- **Moins de 24 Go de RAM, y compris un PC à 16 Go** : `qwen3-coder-30b` si `fits` est vrai. C'est Qwen3-Coder-30B-A3B, environ 19 Go, code et outils, sans réflexion. Carte graphique facultative. Ne pas installer `qwen38-flash-next` ni Mistral Large 4 : le catalogue demande 24 Go pour que Qwen3.8 démarre.
- **24 Go ou plus** : le modèle que `--list` marque `recommended`, sauf si l'utilisateur a demandé Qwen3.8 et que `qwen38-flash-next` a `fits` vrai. Sur une tour à 64 Go et une RTX 3090, c'est le cas utile.
- Si `qwen3-coder-30b` a `status` `needs-ram` ou `needs-disk`, ne pas télécharger. Montrer `reason` et s'arrêter.

## 3. Installer le modèle retenu

Montrer l'identifiant, `download_gb` et le dossier (`~/colibri-models`, ou un autre disque via `--dir` si l'utilisateur le dit). Attendre un oui.

```powershell
py -3 c\coli setup --model qwen3-coder-30b --yes
```

Remplacer l'identifiant par celui retenu à l'étape 2. Ne pas ajouter `--yes` avant le oui de l'utilisateur : lancer d'abord la commande sans `--yes` si le setup pose encore la question, ou n'ajouter `--yes` qu'une fois l'accord donné.

Le setup construit ou télécharge le moteur, télécharge le modèle, démarre Colibri et ouvre le tableau de bord.

Vérifier :

```powershell
py -3 c\coli status
```

Le tableau de bord est `http://127.0.0.1:8000/`. La base OpenAI est `http://127.0.0.1:8000/v1`. Pour arrêter : `py -3 c\coli stop`, ou Ctrl+C dans la fenêtre qui a démarré le serveur.

La fois suivante : `py -3 c\coli serve`, ou `START-HERE.bat` dans le dossier cloné.

## 4. Contrôle

Envoyer une requête courte au modèle annoncé par `http://127.0.0.1:8000/v1/models`. Si la réponse a un `content` vide et un `reasoning_content` rempli, avec `finish_reason` `length`, ce n'est pas une troncature du serveur : la réflexion a mangé `max_tokens`. Relancer avec `enable_thinking` à `false`, ou un plafond plus haut.

Ne pas écouter sur le réseau (`--host 0.0.0.0`) sans une clé fournie par l'utilisateur dans la variable `COLI_API_KEY`.
