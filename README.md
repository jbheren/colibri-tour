# Le local et le frontière

Session filmée depuis un portable, l'agent parle à Colibri sur un PC Windows. Extrait de 12 secondes (à partir de 0:24), pas la vidéo entière. Un GIF joint ne doit pas dépasser 2 à 3 Mo.

![Qwen écrit du Ruby via Colibri](media/qwen-colibri.gif)

Je l'ai accepté : les agents entrent dans notre chaîne de travail. Cela doit se faire de manière prudente. Je ne veux pas d'une chaîne de travail qui dépende entièrement des modèles frontière hébergés.

Pour l'inférence locale, le mur, c'est la mémoire. Faire tenir le modèle entier dans la RAM d'un GPU, dont les prix ont atteint les étoiles.

Colibri, un moteur open source écrit en C, montre une autre voie. Il n'utilise pas le GPU de façon exclusive : la carte garde ce qui est sollicité souvent, la RAM système ce qui doit rester proche, le SSD le reste, lu à la volée. Le point d'équilibre descend. On peut se servir de la machine qu'on a déjà.

Sur mon PC de bureau — une RTX 3090 (24 Go), 64 Go de RAM, un SSD NVMe — un modèle de 68 Go compressés ne tient pas dans la carte. Deux fois trop gros. Avec Colibri, un petit nouveau qui bouscule le secteur, il tourne à environ 35 token/s, et il écrit du Ruby correct, tests compris.

Comment ? Ces modèles sont des Mixture of Experts : à chaque token, seule une partie des paramètres travaille. Colibri ne charge donc pas tout dans la carte. Il garde en mémoire la partie commune. Les experts restent sur le SSD, et il va chercher à la volée ceux dont il a besoin. La carte garde les plus sollicités. La RAM prend le reste de ce qui doit rester proche.

Poussé au bout, le même principe fait tenir Mistral Large 4, « Le Chonk » : 1 000 milliards de paramètres, 233 Go même compressé à l'extrême. Là, la RAM ne suffit plus. Le SSD devient la mémoire. 0,8 token/s, 16 secondes avant que la réponse commence. À ce stade, c'est une expérience, pas un outil. Le simple fait que ce soit possible est déjà une prouesse.

Si le poste de travail local continue d'exister, je pense que notre chaîne de travail sera un mix de modèles locaux et cloud. Les raisons sont multiples, et elles ne motivent pas les mêmes acteurs. Les développeurs veulent pouvoir utiliser leurs outils sans limite. Les organisations cherchent la maîtrise des coûts. Les deux ont besoin de confidentialité. Le mix et le cloisonnement, c'est d'ailleurs la direction prise par Microsoft. GitHub Copilot va choisir, tâche par tâche, un modèle local ou un modèle cloud. Sur les PC Windows RTX Spark, comme le Surface Laptop Ultra, le code peut tourner sur la machine. Les commandes de l'agent passent par MXC, un bac à sable : fichiers, réseau, identifiants, sans ouvrir tout le poste.

Colibri est un très bon symptôme de cette pression. La course à la RAM pour nos machines locales se heurte au mur des prix. Sur l'Apple Store français, le 1er novembre : un Mac Studio M5 Ultra (36 cœurs CPU, 80 cœurs GPU) est à 12 429 € en 256 Go, et à 25 079 € en 512 Go.

Il faut trouver le bon point d'équilibre. Ce type d'outil nous permet de le faire, en ne se limitant pas à la mémoire graphique.

Je partage ici mes découvertes et retours d'expérience, dans une démarche ouverte, sans prosélytisme sur l'usage des agents IA dans mes domaines.

Apprendre, faire, partager.

## Sources

- Colibri (Apache 2.0) : https://github.com/JustVugg/colibri
- Mistral Large 4 : https://mistral.ai/news/mistral-large-4-oss/
- Poids compressés du Chonk (GGUF 1 bit, Unsloth) : https://unsloth.ai/docs/models/mistral-large-4
- Qwen3.8-Flash-Next compressé (IQ2_XS) : https://huggingface.co/ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-GGUF
- Le même principe, sans carte graphique (Marco Franzon, 4 Raspberry Pi 5) : https://x.com/mfranz_on/status/2115884818734911795
- Microsoft, modèles locaux et outils en bac à sable : https://commandline.microsoft.com/local-models-sandboxed-tools-github-windows/
- MXC : https://github.com/microsoft/mxc
- Prix Mac Studio : configurateur Apple Store français, 1er novembre 2026, puce 36 cœurs CPU / 80 cœurs GPU. 12 429 € en 256 Go. 512 Go affiché à + 12 650 €, soit 25 079 €.

## Installation

Si vous utilisez un assistant de code, il peut faire l'installation. Demandez-lui :

> Installe Colibri sur ce Windows en suivant https://raw.githubusercontent.com/jbheren/colibri-tour/main/skills/colibri-tour/SKILL.md

Notes d'installation, pas un fork de Colibri. Deux cas :

- une machine comme la tour de référence (RTX 3090 24 Go, 64 Go de RAM, SSD NVMe) : Qwen3.8-Flash-Next ;
- un PC normal à 16 Go de RAM, carte graphique facultative : Qwen3-Coder-30B-A3B.

Colibri v2.1.0 pour la tour. Le choix 16 Go vient du catalogue Colibri ([README](https://github.com/JustVugg/colibri#which-model-for-my-machine), lu le 1er novembre 2026) : en dessous de 8 Go le modèle ne démarre pas, à partir de 18 Go il tourne comme mesuré. À 16 Go il démarre, une partie des experts peut rester sur le disque. Ne pas installer Qwen3.8-Flash-Next sur 16 Go : le catalogue demande 24 Go pour qu'il démarre.

Ne pas committer de clé d'API. Le serveur refuse une écoute réseau sans clé.

Colibri ne demande pas que le modèle tienne entier dans le GPU. La carte garde les experts chauds, la RAM les couches communes (ou, pour Qwen, les couches denses), le SSD les experts froids.

## Installer (les deux cas)

1. Release Windows sur https://github.com/JustVugg/colibri/releases : `colibri-v*-windows-x86_64.zip` (et l'archive `...-cuda.zip` par-dessus s'il y a une carte Nvidia). Vérifier `SHA256SUMS.txt`. Le lanceur est `coli.cmd`.
2. Avant tout téléchargement de modèle, `coli.cmd setup --list` et ne garder que ce qui tient.
3. La fois suivante, `coli.cmd serve --model <dossier>`. `coli.cmd status` et `coli.cmd stop`.

La tour de référence a été installée ainsi, version épinglée : Python 3.12 (`winget install --id Python.Python.3.12 -e --scope user`), puis les archives `colibri-v2.1.0-windows-x86_64.zip` et `...-windows-x86_64-cuda.zip` (empreintes `SHA256SUMS.txt`) dans `C:\Users\jb\colibri`, puis `python -m pip install -U "huggingface_hub[hf_xet]"`.

## PC 16 Go — Qwen3-Coder-30B-A3B

Modèle de code et d'appels d'outils, sans réflexion. Identifiant : `qwen3-coder-30b`. Téléchargement 19 Go. Mesure publiée par Colibri, processeur seul, tous les experts en RAM : 8,5 à 9,6 token/s. Ce n'est pas la vitesse à attendre à 16 Go.

```bat
coli.cmd setup --model qwen3-coder-30b
```

Carte graphique facultative. Vulkan, y compris une puce intégrée, est pris en charge pour ce modèle. CUDA n'est pas nécessaire. Qwen3.6-35B-A3B (`qwen36-35b`, 23 Go, 10 / 20 Go de RAM) n'est le défaut que si `--list` le marque comme tenant « comme mesuré » ; sur 16 Go, non.

## Mistral Large 4 (« Le Chonk ») — expérience

233 Go, quantification 1 bit (Unsloth `UD-IQ1_S`). Ne tient ni dans le GPU ni dans la RAM.

```powershell
hf download unsloth/Mistral-Large-4-1T-A52B-GGUF --include "UD-IQ1_S/*" --local-dir C:\models\ml4
coli.cmd import C:\models\ml4\UD-IQ1_S\Mistral-Large-4-1T-A52B-UD-IQ1_S-00001-of-00006.gguf
```

L'import est en mode lien : pas de seconde copie.

GPU : Colibri choisit Vulkan pour ce GGUF puis échoue (`COLI_GPU(S) requires COLI_CUDA=1`). Forcer CUDA.

```powershell
$env:COLI_VULKAN = "0"
$env:COLI_CUDA = "1"
coli.cmd serve --ctx 32768 --vram 16 --host 0.0.0.0 --api-key $env:COLI_API_KEY --model C:\models\ml4\UD-IQ1_S\Mistral-Large-4-1T-A52B-UD-IQ1_S.colibri
```

- `--vram 16` : moins d'experts chauds (734 au lieu de 976). Sans ça, 32 768 tokens de contexte ont été refusés une fois le fichier d'usage grossi (56,7 Go prévus pour 51,4 disponibles).
- `--ctx 65536` refuse de démarrer (environ 61 Go prévus). 40 960 démarre, sans marge.
- Un seul moteur à la fois : deux instances ne tiennent pas dans la RAM.
- `coli chat` sans serveur envoie le format de dialogue GLM. Le modèle se présente alors comme Qwen. Passer par `coli serve`, qui applique le gabarit Mistral.

Mesure (CUDA, 36 tokens de prompt, 64 générés, sans réflexion) : 0,77 token/s, lecture du prompt 16,4 s, 41,6 Go de RAM. Ce n'est pas un outil de travail.

## Qwen3.8-Flash-Next — le cas utile

68 Go, `IQ2_XS` (ISTA-DASLab). Trop gros pour la 3090 seule. Tient carte + RAM.

```powershell
hf download ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-GGUF --include "IQ2_XS/*" --local-dir C:\models\q38
coli.cmd import C:\models\q38\IQ2_XS\Qwen3.8-Flash-Next-GSQ-RCO-IQ2_XS-00001-of-00002.gguf
$env:COLI_VULKAN = "1"
coli.cmd serve --ctx 32768 --host 0.0.0.0 --api-key $env:COLI_API_KEY --model C:\models\q38\IQ2_XS\Qwen3.8-Flash-Next-GSQ-RCO-IQ2_XS.colibri
```

Sans `COLI_VULKAN=1`, tout reste sur le processeur (8 token/s). Avec Vulkan : environ 35 token/s, requête comprise, sur une tâche Ruby d'environ 500 tokens (31/10).

Couper la réflexion dans l'appel API (`enable_thinking: false`), sinon le budget de tokens part dans le raisonnement et le code n'est pas émis.

Appel depuis une autre machine du réseau : envoyer `Host: 127.0.0.1`, ou lancer le serveur avec `--allowed-host` pour l'adresse IP. Colibri refuse l'en-tête Host de l'IP publique du LAN par défaut.

## Sources

- https://github.com/JustVugg/colibri
- https://mistral.ai/news/mistral-large-4-oss/
- https://unsloth.ai/docs/models/mistral-large-4
- https://huggingface.co/ISTA-DASLab/Qwen3.8-Flash-Next-GSQ-RCO-GGUF
