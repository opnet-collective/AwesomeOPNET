# Wallets, Keys & Faucet

← [Back to README](../README.md)

---

## OP_WALLET

The official browser extension wallet for OPNET. Required to interact with MotoSwap and other OPNET dApps from a browser.

**Install:** [docs.opnet.org](https://docs.opnet.org) — the installation link is in the getting started section.

Compatible with: Chrome, Brave, and other Chromium-based browsers.

---

## Testnet Faucet

**URL:** https://faucet.opnet.org/

1. Generate a wallet (or use an existing testnet wallet)
2. Copy your P2TR (`bc1p...`) address
3. Paste it at the faucet and request test BTC
4. Confirm receipt with the testnet explorer: https://testnet.opscan.org

---

## The Two-Key System

OPNET wallets have two distinct public keys — a critical difference from Ethereum that every developer needs to understand:

| Key | Derivation | Role |
|-----|------------|------|
| **Tweaked pubkey** | Bitcoin Taproot tweak | Derives the `bc1p...` address shown in explorers and used for UTXOs |
| **Identity key** | Original (un-tweaked) internal key | The `msg.sender` / `Blockchain.tx.sender` value inside contracts |

**The address you see in an explorer is NOT the same value as `msg.sender` in a contract.**

OPNET SDKs handle this transparently. If you write custom transaction signing or calldata encoding, use `btc_getPublicKeyInfo` (or the SDK's `getPublicKeysInfoRaw`) to resolve both keys from an address.

---

## Address Formats

The same identity can appear in two formats:

| Format | Example prefix | Used in |
|--------|---------------|---------|
| **Bech32m** | `op1sq...` or `bc1p...` | Explorers, config files, human-readable contexts |
| **32-byte hex** | `0x000...` | Contract calldata, event `decoded_json`, on-chain references |

Use `btc_getPublicKeyInfo` to convert between them.

---

## Key Types for Transaction Signing

| Key | Format | Purpose |
|-----|--------|---------|
| **WIF private key** | `K...` or `L...` (52 chars) | Standard Bitcoin signing — controls UTXOs |
| **ML-DSA key** | Hex string | Post-quantum signing — required for certain OPNET contract interactions |

Standard environment variable names used across all OPNET tooling:

```bash
WIF_PRIVATE_KEY=your_wif_key
MLDSA=your_mldsa_hex_key
```

ML-DSA (CRYSTALS-Dilithium) is a NIST-standardized post-quantum signature scheme. Generate a keypair using any OPNET SDK or tooling.

---

## Further Reading

- [docs.opnet.org/developers/getting-started/setting-up-environment/for-nodejs](https://docs.opnet.org/developers/getting-started/setting-up-environment/for-nodejs)
- [docs.opnet.org/bitcoin-address-system](https://docs.opnet.org/bitcoin-address-system)
- [docs.opnet.org/developers/walletconnect/setup](https://docs.opnet.org/developers/walletconnect/setup)
