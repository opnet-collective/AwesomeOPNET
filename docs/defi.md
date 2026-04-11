# DeFi on OPNET

← [Back to README](../README.md)

OPNET launched with two live DEXes on mainnet (March 19, 2026): **NativeSwap** and **Motoswap**. Together they form the core DeFi primitive layer — NativeSwap for BTC↔Token, Motoswap for Token↔Token.

---

## NativeSwap

**The BTC-native AMM.** Direct BTC ↔ OP-20 token swaps with no wrapped BTC required.

- **Website / docs:** [github.com/btc-vision/native-swap](https://github.com/btc-vision/native-swap)
- **Mainnet address:** `op1sqzjrvunc39nahkr6a0d4vx7rjdfjfc7qtqhp9wks`

### Architecture

A **single contract** manages all BTC/OP-20 token pools. There is no per-token "pair address" as in Uniswap. Each pool is identified by its OP-20 token address within the NativeSwap contract.

### The 2-Phase Swap

NativeSwap requires **two Bitcoin transactions** to complete any swap. This is fundamental to how native BTC movement works on Bitcoin L1.

**Phase 1 — Reservation:**
- Caller broadcasts a Bitcoin tx with the swap reservation calldata
- The swap price is locked for blocks **N+1 through N+5** (a 5-block window)
- A reservation fee is paid upfront

**Phase 2 — Execution (Settlement):**
- Within the 5-block window, the caller broadcasts a second Bitcoin tx
- Actual BTC moves; tokens transfer
- Settlement can happen at **any point in the window** — not just at block N+2

**Why 2-phase?**
Moving native L1 satoshis requires a UTXO commitment. The first transaction commits to the swap; the second transaction moves the actual Bitcoin value. This cannot be collapsed into a single atomic transaction on Bitcoin — it is a property of the base layer, not a design choice.

**The 5-block window as optionality:**
The reservation is valid for 5 blocks. A sophisticated trader can monitor price movement during the window and settle at the optimal moment rather than immediately.

### NativeSwap Events

| Event | When emitted |
|-------|-------------|
| `LiquidityListed` | First liquidity deposit for a token — pool created |
| `LiquidityAdded` | Additional liquidity added to an existing pool |
| `Swapped` | Swap completed — reserve snapshot updated |

### V1 Single-LP Risk

In NativeSwap V1, each BTC/Token pool has exactly **one LP — the creator**. The creator can remove their liquidity at any time, including during a pending reservation window. If liquidity is removed mid-reservation, the settlement transaction will fail or yield near-zero tokens, and the reservation fee is lost.

This is a known property of the V1 design. Track creator behavior when interacting with pools programmatically.

---

## Motoswap

**The OP-20/OP-20 AMM.** Uniswap V2–style DEX for token-to-token swaps on OPNET.

- **Website:** https://motoswap.org
- **GitHub:** [github.com/btc-vision/motoswap](https://github.com/btc-vision/motoswap)
- **Factory address:** `op1sqpud62qd6gewyz4vd8x6lyrtwv6dkrd8n56j5ujw`

### Architecture

The factory contract creates one **pair contract** per Token/Token pair. Each pair maintains its own reserves and handles swaps for that specific pair.

**MOTO as hub token:** All meaningful Motoswap liquidity routes through the MOTO token. Every token pair is a `Token/MOTO` pair. A `TokenA → TokenB` swap routes through `TokenA → MOTO → TokenB`. This is the same model as WBNB on BSC or WETH on Uniswap V2.

- **MOTO token address:** `0x0a6732489a31e6de07917a28ff7df311fc5f98f6e1664943ac1c3fe7893bdab5`

### Single-Phase Swap

Motoswap operates in a **single Bitcoin transaction** — no reservation phase. This is because no native BTC moves; the swap operates entirely on OP-20 token balances within the OPNET execution environment. A single tx carries the calldata for `transferFrom → swap → transfer`.

### Fee Model

- **0.2% fee** on every Token/Token swap
- Fees are distributed to MOTO stakers via MotoChef
- With a MOTO stake, part of swap fees are returned to you as staking yield

### Motoswap Events

| Event | Emitted by | When |
|-------|-----------|------|
| `PoolCreated` | Factory | New pair contract deployed |
| `Synced` | Pair | Every reserve change (swap, add/remove liquidity) |

---

## MotoChef — Staking & Farming

The yield farming contract for Motoswap LP tokens.

- **Address:** `op1sqq9axjh9mm03sz962n5dvgfae0pnvwr0zvzhm28r`

LP token holders stake their Motoswap LP tokens into MotoChef to earn MOTO emissions.

**Key metrics:**
- `motoPerBlock` — MOTO emission rate per block (controls APR)
- `totalAllocPoint` — total allocation points across all farm pools (determines share per pool)

### Events

| Event | Description |
|-------|-------------|
| `PoolAdded` | New farm pool registered |
| `PoolUpdated` | Farm pool allocation changed |

---

## BTC → Token → MOTO → BTC Arbitrage Cycle

The interaction between NativeSwap and Motoswap enables a canonical 3-hop arbitrage cycle:

```
BTC → [NativeSwap 2-phase] → TokenX → [Motoswap single-tx] → MOTO → [NativeSwap 2-phase] → BTC
```

5 total transactions, ~4 blocks of price exposure between reservation and final settlement. This is the primary arb structure in the OPNET DeFi ecosystem.

---

## DeFi Contracts at a Glance

| Contract | Address | GitHub |
|----------|---------|--------|
| NativeSwap | `op1sqzjrvunc39nahkr6a0d4vx7rjdfjfc7qtqhp9wks` | [btc-vision/native-swap](https://github.com/btc-vision/native-swap) |
| Motoswap Factory | `op1sqpud62qd6gewyz4vd8x6lyrtwv6dkrd8n56j5ujw` | [btc-vision/motoswap](https://github.com/btc-vision/motoswap) |
| MotoChef | `op1sqq9axjh9mm03sz962n5dvgfae0pnvwr0zvzhm28r` | — |
| MOTO Token | `0x0a6732489a31e6de07917a28ff7df311fc5f98f6e1664943ac1c3fe7893bdab5` | — |
