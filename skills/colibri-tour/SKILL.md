---
name: colibri-tour
description: Faire tourner ou interroger Colibri sur la tour Windows (RTX 3090, 64 Go) — Qwen utile, Mistral Large 4 en expérience. Ne pas committer la clé API.
---

# Colibri sur la tour

Lire `README.md` à la racine de ce dépôt pour l'installation et les commandes exactes.

## Quand l'utiliser

- Un agent doit coder ou répondre via le modèle local de la tour, pas via un modèle hébergé.
- Il faut relancer Colibri, choisir le bon GPU (CUDA ou Vulkan), ou expliquer une réponse tronquée.

## Règles

- Un seul modèle servi à la fois.
- Clé : variable `COLI_API_KEY`. Ne jamais l'écrire dans un fichier versionné.
- Qwen : `COLI_VULKAN=1`. Mistral Large 4 : `COLI_VULKAN=0` et `COLI_CUDA=1`, plus `--vram 16` si le contexte de 32 768 est refusé.
- Dans l'API chat, passer `enable_thinking: false` pour Qwen. Sans ça, `finish_reason: length` et un `content` vide ne veulent pas dire que le serveur a tronqué la réponse : le budget est parti dans la réflexion.
- Depuis une autre machine, `Host: 127.0.0.1` si l'appel se fait par l'IP du LAN, sauf `--allowed-host` ajouté au serveur.
- Mistral Large 4 à ~0,8 token/s est une expérience. Ne pas le proposer comme modèle de travail.
