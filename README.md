# signtool

> A command-line tool for **digital file signing and verification** using RSA 2048 + SHA-256 (PKCS1v15).

```
signtool keygen --name mykey
signtool sign   --key mykey_private.pem --file report.pdf
signtool verify --key mykey_public.pem  --file report.pdf
signtool info   report.pdf.sig
```

---

## Features

| Feature | Details |
|---|---|
| Algorithm | RSA (1024 / 2048 / 4096 bits) + SHA-256, PKCS#1 v1.5 padding |
| Key format | PEM (standard OpenSSL format) |
| Signature file | `.sig` — JSON with base64 signature, algorithm, timestamp, SHA-256 hash |
| Private key protection | Optional AES passphrase encryption |
| Batch signing | Glob patterns (`docs/*.pdf`) |
| Output | Colored tables, progress spinners, and panels via **rich** |
| Exit codes | `0` = valid / success, `1` = invalid / error |

---

## Installation

### Requirements

- Python >= 3.8
- pip

### From source (recommended)

```bash
cd signtool
pip install -e .
```

This registers the `signtool` command globally.

### Dependencies only

```bash
pip install -r requirements.txt
```

---

## Commands

### `signtool keygen` — Generate an RSA key pair

```
signtool keygen [OPTIONS]
```

| Option | Default | Description |
|---|---|---|
| `--bits` | `2048` | Key size: `1024`, `2048`, or `4096` |
| `--output-dir`, `-o` | `.` | Directory to save PEM files |
| `--name`, `-n` | `key` | Base name for the files |
| `--passphrase`, `-p` | *(none)* | Encrypt the private key with a passphrase |

**Output files:**

- `{name}_private.pem` — Keep this **secret** (permissions set to `600` on POSIX)
- `{name}_public.pem` — Share this freely

**Examples:**

```bash
# Default 2048-bit key pair named "key"
signtool keygen

# 4096-bit key pair in a specific directory
signtool keygen --bits 4096 --output-dir ~/keys --name myproject

# Password-protected private key
signtool keygen --name mykey --passphrase "s3cr3tPassphrase"
```

---

### `signtool sign` — Sign one or more files

```
signtool sign --key PRIVATE_KEY --file FILE [OPTIONS]
```

| Option | Required | Description |
|---|---|---|
| `--key`, `-k` | Yes | Path to the RSA private key PEM file |
| `--file`, `-f` | Yes (multiple) | File(s) to sign; repeatable; supports glob patterns |
| `--output-dir`, `-o` | No | Directory for `.sig` files (default: same as each file) |
| `--passphrase`, `-p` | No | Passphrase if the private key is encrypted |

Each file `foo.bar` produces a companion `foo.bar.sig` containing:

```json
{
  "version": 1,
  "tool": "signtool v1.0.0",
  "algorithm": "RSA-SHA256-PKCS1v15",
  "timestamp": "2024-01-15T10:30:00.000000+00:00",
  "filename": "foo.bar",
  "sha256": "e3b0c44298fc1c149afb...",
  "signature": "<base64-encoded RSA signature>"
}
```

**Examples:**

```bash
# Sign a single file
signtool sign --key mykey_private.pem --file report.pdf

# Sign multiple files explicitly
signtool sign --key mykey_private.pem \
              --file doc1.pdf \
              --file doc2.pdf \
              --file contract.docx

# Sign all PDFs in a directory (glob pattern)
signtool sign --key mykey_private.pem --file "docs/*.pdf"

# Sign and store .sig files in a separate directory
signtool sign --key mykey_private.pem \
              --file "build/*.zip" \
              --output-dir signatures/

# Sign with an encrypted key
signtool sign --key mykey_private.pem \
              --file important.pdf \
              --passphrase "mySecret"
```

---

### `signtool verify` — Verify a file's signature

```
signtool verify --key PUBLIC_KEY --file FILE [--sig SIG_FILE]
```

| Option | Required | Description |
|---|---|---|
| `--key`, `-k` | Yes | Path to the RSA public key PEM file |
| `--file`, `-f` | Yes | File to verify |
| `--sig`, `-s` | No | Path to the `.sig` file (default: `{file}.sig`) |

**Exit codes:**

- `0` — Signature is **valid**; the file has not been tampered with
- `1` — Signature is **invalid** or an error occurred

**Examples:**

```bash
# Verify using the default .sig path (report.pdf.sig)
signtool verify --key mykey_public.pem --file report.pdf

# Specify the .sig file explicitly
signtool verify --key mykey_public.pem \
                --file report.pdf \
                --sig  signatures/report.pdf.sig

# Use in a script (exit code 0 = valid)
if signtool verify --key pub.pem --file archive.zip; then
    echo "Archive is authentic."
else
    echo "WARNING: archive has been tampered with!"
fi
```

---

### `signtool info` — Display signature metadata

```
signtool info SIG_FILE
```

Prints a rich formatted panel with all metadata stored in the `.sig` file.

**Example:**

```bash
signtool info report.pdf.sig
```

**Sample output:**

```
 Signature Metadata — report.pdf.sig
 ╭────────────────────┬────────────────────────────────────────────────╮
 │ Field              │ Value                                          │
 ├────────────────────┼────────────────────────────────────────────────┤
 │ Sig file           │ /home/user/report.pdf.sig                      │
 │ Format ver         │ 1                                              │
 │ Tool               │ signtool v1.0.0                                │
 │ Algorithm          │ RSA-SHA256-PKCS1v15                            │
 │ Filename           │ report.pdf                                     │
 │ Signed at          │ 2024-01-15T10:30:00.000000+00:00               │
 │ SHA-256            │ e3b0c44298fc1c149afbf4c8996fb924…              │
 │ Signature          │ AbCdEf12…                                      │
 │ Sig length         │ 344 chars (base64)                             │
 ╰────────────────────┴────────────────────────────────────────────────╯
```

---

## Full Workflow Example

```bash
# 1. Generate keys
signtool keygen --bits 2048 --output-dir ./keys --name release

# 2. Sign your release artifacts
signtool sign --key keys/release_private.pem \
              --file "dist/*.tar.gz" \
              --output-dir dist/signatures/

# 3. Share the public key + signatures with your users

# 4. Users verify the download
signtool verify --key release_public.pem \
                --file myapp-1.0.tar.gz \
                --sig  signatures/myapp-1.0.tar.gz.sig

# 5. Inspect a .sig file for details
signtool info signatures/myapp-1.0.tar.gz.sig
```

---

## Demo Scripts

Run the complete demo to see all commands in action:

**Windows:**

```bat
demo.bat
```

**Linux / macOS / WSL:**

```bash
chmod +x demo.sh
./demo.sh
```

The demo:
1. Installs `signtool` in editable mode
2. Creates sample text files
3. Generates RSA-2048 and RSA-4096 key pairs
4. Signs individual and multiple files
5. Inspects a `.sig` file
6. Verifies a valid signature ✓
7. Demonstrates tamper detection (modifies a file and shows INVALID) ✗

---

## Project Structure

```
signtool/
├── README.md            # This file
├── requirements.txt     # Python dependencies
├── setup.py             # Package & entry point definition
├── demo.sh              # Bash demo script
├── demo.bat             # Windows demo script
└── signtool/
    ├── __init__.py      # Version & package metadata
    ├── cli.py           # Click CLI commands (keygen, sign, verify, info)
    ├── keygen.py        # RSA key pair generation
    ├── signer.py        # File signing logic
    └── verifier.py      # Signature verification logic
```

---

## Security Notes

- **Private key**: Never share your `_private.pem` file. Use `--passphrase` to add an extra layer of protection.
- **Key size**: RSA-2048 is the minimum recommended size. Use RSA-4096 for long-term security.
- **Algorithm**: SHA-256 + PKCS#1 v1.5 is widely supported and suitable for most use cases. For new systems, consider migrating to PSS padding in the future.
- **Timestamps**: The timestamp in the `.sig` file is informational only (from the signing machine's clock) — it is **not** a trusted timestamp authority.

---

## Dependencies

| Package | Version | Purpose |
|---|---|---|
| `click` | >= 8.1.0 | CLI framework |
| `cryptography` | >= 41.0.0 | RSA key generation & signing |
| `rich` | >= 13.0.0 | Colored terminal output |

---

## License

MIT License — see [LICENSE](LICENSE) for details.
#   s e c W e b M a i l  
 #   s e c W e b M a i l  
 #   S W M  
 