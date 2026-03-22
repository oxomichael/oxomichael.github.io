---
title: "🚀 Guide d'Installation IA Locale : llama.cpp + Docker + OpenCode (AMD Edition)"
date: "2026-03-22T10:00:00+02:00"
draft: false
translationKey: "ai-local-llamacpp-opencode"
---

Ce guide permet de configurer un environnement de développement IA exploitant l'accélération matérielle ROCm sur un processeur AMD Ryzen AI 7 avec iGPU Radeon 860M.

💻 Caractéristiques de la Machine cible

    CPU : AMD Ryzen AI 7 350
    GPU : Radeon 860M (Architecture RDNA 3.5 / gfx1150)
    RAM : 32 Go (Partagée avec l'iGPU)
    OS : Ubuntu 24.04+ (Recommandé)

# 1. Structure des dossiers
```
/home/ton-user/ai-lab/
├── models/             # Dossier où tu places tes fichiers .gguf
└── compose.yaml        # Fichier de configuration Docker
```

# 2. Le fichier compose.yaml (Mode Dual: GPU & CPU)

Ce fichier définit deux services. Tu pourras choisir celui que tu veux lancer.
```YAML
services:
  # --- Version GPU (Radeon 860M) ---
  # Utilise l'image optimisée pour ROCm (AMD)
  llamacpp-gpu:
    image: ghcr.io/ggml-org/llama.cpp:server-rocm
    container_name: llamacpp-gpu
    restart: unless-stopped
    ports:
      - "8080:8080"
    devices:
      - /dev/kfd:/dev/kfd
      - /dev/dri:/dev/dri
    group_add:
      - video
      - render
    volumes:
      - ./models:/models
    environment:
      # Pour forcer la reconnaissance de l'iGPU Strix Point
      - HSA_OVERRIDE_GFX_VERSION=11.0.0
    command: >
      -m /models/qwen2.5-coder-7b-instruct-q4_k_m.gguf
      --host 0.0.0.0
      --port 8080
      --n-gpu-layers 100
      --ctx-size 32768
      --flash-attn
      --metrics

  # --- Version CPU (Standard) ---
  llamacpp-cpu:
    image: ghcr.io/ggml-org/llama.cpp:server
    container_name: llamacpp-cpu
    ports:
      - "8081:8080"
    volumes:
      - ./models:/models
    command: >
      -m /models/qwen2.5-coder-7b-instruct-q4_k_m.gguf
      --host 0.0.0.0
      --port 8080
      --ctx-size 32768
      --threads 8
```

# 3. Utilisation

Pour lancer la version GPU (recommandé) :
```Bash
docker compose up -d llamacpp-gpu
```

Pour voir si l'iGPU est bien utilisé (logs) :
```Bash
docker logs -f llamacpp-gpu
```

    Cherche la ligne : ggml_roc_init: found 1 ROCm devices

4. Configuration d'OpenCode (opencode.json)

On pointe vers le port 8080 (GPU) ou 8081 (CPU) selon ton besoin.
```JSON
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "llamacpp-docker": {
      "name": "Llama.cpp Docker",
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "http://127.0.0.1:8080/v1"
      },
      "models": {
        "qwen2.5-coder-7b": {
          "name": "qwen2.5-coder-7b"
        }
      }
    }
  },
  "model": "llamacpp-docker/qwen2.5-coder-7b"
}
```
