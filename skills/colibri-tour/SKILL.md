---
name: colibri-tour
description: Installer Colibri sur Windows et un modèle qui tient — Qwen3.8-Flash-Next sur une tour 64 Go, Qwen3-Coder-30B sur un PC 16 Go. Ne pas committer la clé API.
---

# Colibri sur Windows

Lire `README.md` à la racine de ce dépôt. Demander avant de télécharger un modèle ou d'installer un paquet système.

## Choisir le modèle

Regarder la RAM, puis `START-HERE.bat --list`.

- **16 Go, PC normal, GPU facultatif** : `qwen3-coder-30b` (Qwen3-Coder-30B-A3B, 19 Go sur disque). Catalogue Colibri : ne démarre pas sous 8 Go, tourne comme mesuré à partir de 18 Go. À 16 Go, une partie des experts peut venir du disque. Ne pas proposer Qwen3.8-Flash-Next (24 Go minimum annoncés) ni Mistral Large 4.
- **64 Go et RTX 3090, comme la tour de référence** : Qwen3.8-Flash-Next IQ2_XS, procédure manuelle du README. Environ 35 token/s avec `COLI_VULKAN=1`. Mistral Large 4 seulement si on demande l'expérience (0,8 token/s, CUDA forcé, `--vram 16`).
- Entre les deux : ne pas deviner. Faire confiance à `--list`, et dire pourquoi un modèle est écarté.

## Installer

1. ZIP de https://github.com/JustVugg/colibri, `START-HERE.bat`.
2. `START-HERE.bat --list`, puis `START-HERE.bat --model qwen3-coder-30b` sur 16 Go, ou la procédure Qwen du README sur la tour.
3. Un seul modèle servi à la fois.

## Après l'installation

- Clé : variable `COLI_API_KEY` si le serveur écoute le réseau. Ne jamais l'écrire dans un fichier versionné. En local, le tableau de bord est `http://127.0.0.1:8000/`.
- Qwen3.8 sur la tour : `COLI_VULKAN=1`. Mistral Large 4 : `COLI_VULKAN=0` et `COLI_CUDA=1`.
- Appel API chat de Qwen3.8 : `enable_thinking: false`. Sinon `finish_reason: length` et un `content` vide veulent dire que le budget est parti dans la réflexion, pas que le serveur a tronqué tout seul.
- Depuis une autre machine, `Host: 127.0.0.1` si l'appel se fait par l'IP du LAN, sauf `--allowed-host` au lancement.
