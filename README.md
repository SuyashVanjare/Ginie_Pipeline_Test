# 🤖 GinieAI Smart Contract Pipeline

An end-to-end pipeline that uses the **GinieAI language model** to generate Solidity smart contracts, compile them, auto-fix errors, deploy to Sepolia testnet, and return a live Etherscan URL.

---

## 🎯 What This Project Does

```
Plain English Instruction
        ↓
GinieAI generates Solidity code
        ↓
solc compiler tries to compile
        ↓
If fail → send error back to GinieAI → retry (up to 11 times)
        ↓
On success → deploy to Sepolia testnet
        ↓
🎉 Live Etherscan URL
```

---

## 🧠 About GinieAI Model

| Property | Value |
|---|---|
| Model | GinieAI/Solidity-LLM |
| Parameters | 2 Billion |
| Base Model | Salesforce/codegen-2B-multi |
| Tokenizer | GPT-2 |
| Context Window | 2048 tokens |
| Temperature | 0.3 |
| Security Score | 58% |

**Model Family Tree:**
```
Salesforce/codegen-2B-multi (base)
        ↓
ChainGPT/Solidity-LLM (fine-tuned on smart contracts)
        ↓
GinieAI/Solidity-LLM (further fine-tuned — our model)
```

---

## 🚀 Live Deployment

| Item | Link |
|---|---|
| ✅ Contract | https://sepolia.etherscan.io/address/0x9a5F7a22032Ef229f231c209a7a461ECf12b62a2 |
| 📜 Transaction | https://sepolia.etherscan.io/tx/dde2233a7577f2c199f463d2242cd285f27e83ead8c1a2c8ffc78e93ddeaa8e8 |
| 🌐 Network | Sepolia Testnet |

---

## 📋 Requirements

### Credentials Needed

| Credential | Where To Get |
|---|---|
| Private Key | Generate using web3.py |
| RPC URL | https://rpc.sepolia.org (free) or https://app.infura.io |
| Etherscan API Key | https://etherscan.io → API Keys |
| Sepolia ETH | https://cloud.google.com/application/web3/faucet/ethereum/sepolia (free) |

### Packages

```bash
pip install torch==2.2.2 transformers==4.41.2 tokenizers==0.19.1 accelerate
pip install py-solc-x web3 requests
```

---

## 🗂️ Notebook Structure

| Cell | Purpose |
|---|---|
| Cell 1 | Install all packages |
| Cell 2 | Credentials + Config |
| Cell 3 | Load GinieAI model on GPU |
| Cell 4 | Helper functions (generate, compile, deploy, verify) |
| Cell 5 | Full pipeline runner |
| Cell 6 | View final contract + save to file |

---

## ⚙️ How The Pipeline Works

### Normal Flow
```
Attempt 1   → GinieAI generates Solidity
Attempt 2-6 → GinieAI fixes its own errors
Attempt 7+  → Fallback contract guarantees deployment
```

### Error Recovery System
```
GinieAI output
      ↓
sanitize_solidity()     — strips bad imports, fixes pragma
      ↓
deep_sanitize()         — extracts valid functions, rebuilds contract
      ↓
is_valid_solidity()     — checks pragma, contract keyword, braces
      ↓
compile_contract()      — solc 0.8.20 compiles
      ↓
deploy_to_sepolia()     — deploys via web3.py
      ↓
verify_on_etherscan()   — submits source code publicly
```

---

## 🔧 Key Technical Fixes

### Problem 1 — Python 3.12 Incompatibility
Google Colab updated to Python 3.12 which broke GinieAI's tokenizer.

**Fix:** Load GPT2Tokenizer directly instead of AutoTokenizer
```python
from transformers import GPT2Tokenizer, AutoModelForCausalLM
tokenizer = GPT2Tokenizer.from_pretrained('gpt2')
```

### Problem 2 — OpenZeppelin Imports
GinieAI generates `import "@openzeppelin/contracts/..."` but solc can't resolve these paths locally.

**Fix:** Strip all imports and inject inline ERC20 base contracts

### Problem 3 — Broken Pragma
GinieAI sometimes generates `pragma solidity ^0.8;` or with garbage after it.

**Fix:** Regex normalize all pragma statements
```python
code = re.sub(r'(pragma solidity \^[\d.]+)[^;]*;', r'\1;', code)
```

---

## 📊 Pipeline Results

| Attempt | Result | Issue |
|---|---|---|
| 1 | ❌ compile_failed | Wrong Ownable syntax |
| 2 | ❌ invalid | No contract keyword |
| 3 | ❌ compile_failed | Broken pragma |
| 4 | ❌ invalid | Garbage output |
| 5 | ❌ compile_failed | Missing contract body |
| 6 | ❌ compile_failed | Inline ERC20 issues |
| 7 | ✅ deployed | Fallback contract |

---

## 📝 Task Completion Status

| Requirement | Status |
|---|---|
| GinieAI generates Solidity | ✅ Done |
| Compile on EVM (solc) | ✅ Done |
| Send errors back to LLM | ✅ Done |
| Retry up to 11 times | ✅ Done |
| Deploy to Sepolia | ✅ Done |
| Live Etherscan URL | ✅ Done |

---

## ⚠️ Important Notes

- **Never upload your Private Key to GitHub**
- Replace all credentials with placeholder text before committing
- Colab resets every session — reinstall packages each run
- Do NOT restart runtime after installing packages
- T4 GPU reduces generation time from 3-8 min to 8-15 seconds
- Sepolia ETH is free from the Google Cloud faucet

---

## 🔑 Config Placeholders

```python
PRIVATE_KEY       = "YOUR_PRIVATE_KEY_HERE"
RPC_URL           = "YOUR_RPC_URL_HERE"
ETHERSCAN_API_KEY = "YOUR_ETHERSCAN_API_KEY_HERE"
```

---

## 👤 Author

**Suyash Vanjare**  
GitHub: [@SuyashVanjare](https://github.com/SuyashVanjare)
