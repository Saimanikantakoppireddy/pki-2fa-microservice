# PKI + TOTP 2FA Microservice

This project is a secure, containerized microservice that demonstrates enterprise-grade security practices using:

- RSA 4096-bit Public Key Infrastructure (PKI)
- Time-based One-Time Passwords (TOTP) for 2FA
- Docker + cron + volumes for production-style deployment

The microservice exposes three REST API endpoints to:

1. Decrypt an encrypted seed using RSA/OAEP with SHA-256
2. Generate TOTP 2FA codes from the decrypted seed
3. Verify provided TOTP codes with a ±1 time window

Cron runs inside the container every minute to log the current 2FA code to a persistent volume.

---

## Features

- **RSA 4096-bit key pair**
  - Public exponent: **65537**
  - Keys stored as PEM: `student_private.pem`, `student_public.pem`
- **Seed decryption**
  - Algorithm: **RSA/OAEP**
  - Hash: **SHA-256**
  - MGF: **MGF1(SHA-256)**
  - Label: **None**
- **TOTP (2FA)**
  - Algorithm: **SHA-1**
  - Period: **30 seconds**
  - Digits: **6**
  - Hex seed (64 chars) → bytes → Base32 → TOTP
  - Verification tolerance: **±1 period** (±30s)
- **Dockerized**
  - Multi-stage build (builder + runtime)
  - Runs **FastAPI** on port **8080**
  - Runs **cron** daemon for periodic logging
  - Timezone: **UTC**
- **Persistent volumes**
  - `/data/seed.txt` – decrypted seed (survives container restarts)
  - `/cron/last_code.txt` – cron-logged TOTP codes

---

## Tech Stack

- **Language:** Python 3.11
- **Framework:** FastAPI + Uvicorn
- **Crypto Library:** `cryptography`
- **TOTP Library:** `pyotp`
- **HTTP Client (for seed request):** `requests`
- **Containerization:** Docker, Docker Compose
- **Scheduler:** cron (inside container)

---

## Project Structure

```text
pki-2fa-microservice/
├─ app/
│  ├─ __init__.py
│  ├─ main.py           # FastAPI app + API endpoints
│  ├─ crypto_utils.py   # RSA key loading, seed decryption, validation
│  └─ totp_utils.py     # TOTP generation, verification, validity timer
├─ scripts/
│  ├─ generate_keys.py          # RSA key generation (4096 bits, e=65537)
│  ├─ request_seed.py           # Calls instructor API, saves encrypted_seed.txt
│  ├─ test_decrypt_endpoint.py  # Local test for /decrypt-seed
│  ├─ test_verify_2fa.py        # Local test for /generate-2fa + /verify-2fa
│  └─ log_2fa_cron.py           # Cron script: logs TOTP code every minute
├─ cron/
│  └─ 2fa-cron          # cron configuration (runs log_2fa_cron.py every minute)
├─ student_private.pem   # Student private key (RSA 4096)
├─ student_public.pem    # Student public key
├─ instructor_public.pem # Instructor public key (provided by course)
├─ encrypted_seed.txt    # (ignored by Git) encrypted seed from instructor API
├─ Dockerfile
├─ docker-compose.yml
├─ requirements.txt
├─ .gitignore
├─ .gitattributes
└─ README.md
