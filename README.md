# Colibri sur une RTX 3090 (24 Go) et 64 Go de RAM

Notes d'installation et d'exécution, pas un fork de Colibri. Machine : PC Windows, RTX 3090 24 Go, 62 Go de RAM utiles, SSD NVMe (Samsung 9100 PRO 2 To). Colibri v2.1.0. Mesures du 31 octobre 2026.

Colibri ne demande pas que le modèle tienne entier dans le GPU. La carte garde les experts chauds, la RAM les couches communes (ou, pour Qwen, les couches denses), le SSD les experts froids.

Ne pas committer de clé d'API. Le serveur refuse une écoute réseau sans clé.

## Installer

1. Python 3.12 : `winget install --id Python.Python.3.12 -e --scope user`
2. Releases GitHub de [colibri](https://github.com/JustVugg/colibri) : `colibri-v2.1.0-windows-x86_64.zip`, puis l'archive `...-windows-x86_64-cuda.zip` par-dessus. Vérifier `SHA256SUMS.txt`. Décompresser dans `C:\Users\jb\colibri`.
3. `python -m pip install -U "huggingface_hub[hf_xet]"`

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
