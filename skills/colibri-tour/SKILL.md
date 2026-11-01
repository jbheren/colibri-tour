---
name: colibri-tour
description: Install and start Colibri on the user's Windows machine. Choose Qwen3-Coder-30B on 16 GB of RAM, Qwen3.8-Flash-Next only if the machine has at least 24 GB. Ask before any download.
---

# Install Colibri on this Windows machine

This page is for the assistant asked to run it. A person can follow it too. Install from the Windows release, not from a clone of the source repository.

## Rules

- Ask before downloading a model (19 GB and up) and before `winget` or any other system package.
- Do not invent an API key and do not write one into a file. For local use, do not open the port.
- Serve one model at a time.
- If a command stops, run it again: the download resumes.

## 1. Colibri

The launcher is `coli.cmd`, next to `colibri.exe`.

If a folder with `coli.cmd` already exists (often `C:\Users\<user>\colibri`), use it. Otherwise download the latest Windows release from https://github.com/JustVugg/colibri/releases (`colibri-v*-windows-x86_64.zip`, and `...-cuda.zip` on top if there is an Nvidia card). Check `SHA256SUMS.txt`, unzip, and work from that folder.

If `py -3` is missing, show `winget install -e --id Python.Python.3.12 --scope user` and run it only after a yes.

## 2. What the machine can hold

From the folder that contains `coli.cmd`:

```powershell
coli.cmd setup --list --json
```

Read `memory.total`. Tell the user which model you kept, its size, and why the others were set aside.

- **Under 24 GB of RAM, including a 16 GB PC**: `qwen3-coder-30b` if `fits` is true. That is Qwen3-Coder-30B-A3B, about 19 GB, code and tools, no thinking. A graphics card is optional. Do not install `qwen38-flash-next` or Mistral Large 4: the catalog asks for 24 GB before Qwen3.8 starts.
- **24 GB or more**: the model `--list` marks `recommended`, unless the user asked for Qwen3.8 and `qwen38-flash-next` has `fits` true. On a 64 GB tower with an RTX 3090, that is the useful case.
- If `qwen3-coder-30b` has `status` `needs-ram` or `needs-disk`, do not download. Show `reason` and stop.

## 3. Install the chosen model

Show the id, `download_gb`, and the folder (`~/colibri-models`, or another disk via `--dir` if the user says so). Wait for a yes.

```powershell
coli.cmd setup --model qwen3-coder-30b
```

Replace the id with the one chosen in step 2. Add `--yes` only after agreement.

Setup downloads the model, starts Colibri, and opens the dashboard.

Check:

```powershell
coli.cmd status
```

The dashboard is `http://127.0.0.1:8000/`. The OpenAI base URL is `http://127.0.0.1:8000/v1`. To stop: `coli.cmd stop`, or Ctrl+C in the window that started the server.

Next time: `coli.cmd serve --model <model folder>`, from the same folder.

## 4. Check

Send a short request to the model announced by `http://127.0.0.1:8000/v1/models`. If the reply has an empty `content` and a filled `reasoning_content`, with `finish_reason` `length`, the server did not truncate on its own: thinking used up `max_tokens`. Retry with `enable_thinking` set to `false`, or a higher cap.

Do not listen on the network (`--host 0.0.0.0`) without a key the user put in `COLI_API_KEY`.
