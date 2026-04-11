# Token Standards

← [Back to README](../README.md)

---

## OP-20 — Fungible Tokens

The primary fungible token standard on OPNET. Conceptually equivalent to ERC-20 with important behavioral differences.

**Base class:** `OP20` from `@btc-vision/btc-runtime`

### Key Differences from ERC-20

| Aspect | ERC-20 | OP-20 |
|--------|--------|-------|
| Approval | `approve(spender, amount)` | `increaseAllowance(spender, amount)` |
| Reduce approval | `approve(spender, 0)` | `decreaseAllowance(spender, amount)` |
| Arithmetic | Native Solidity overflow checks (0.8+) | `SafeMath` required — raw u256 operators silently overflow |
| Deployment | Single tx | Two-phase Taproot (funding + reveal) |

**Why `increaseAllowance` instead of `approve`?**
The standard ERC-20 `approve` pattern has a known race condition: a spender can front-run an approval change to spend both the old and new allowance. OP-20 eliminates this by using delta-based allowance changes.

### OP-20 Interface (key methods)

| Method | Description |
|--------|-------------|
| `name()` | Token name |
| `symbol()` | Token symbol |
| `decimals()` | Decimal places |
| `totalSupply()` | Current total supply |
| `balanceOf(address)` | Balance of an address |
| `transfer(to, amount)` | Transfer tokens |
| `transferFrom(from, to, amount)` | Spend from allowance |
| `increaseAllowance(spender, amount)` | Increase spending allowance |
| `decreaseAllowance(spender, amount)` | Decrease spending allowance |
| `allowance(owner, spender)` | Current allowance |

### Reference Implementations

- [btc-vision/example-tokens](https://github.com/btc-vision/example-tokens) — official example contracts
- [docs.opnet.org](https://docs.opnet.org) — OP-20 documentation

---

## OP-721 — Non-Fungible Tokens

NFT standard equivalent to ERC-721.

**Base class:** `OP_NET` from `@btc-vision/btc-runtime` (no dedicated `OP721` base class — implement storage maps manually)

Token ownership is tracked using `Map<u256, Address>` (token ID → owner) and balances via `Map<Address, u256>`.

See the [contracts guide](contracts.md) for an implementation example.

---

## OP-20S — Stablecoins (Upcoming)

Extension of OP-20 designed for stablecoins with additional compliance features.

**Status:** Expected on OPNET mainnet Q2 2026. First tokens: USDT and USDC on Bitcoin.

When OP-20S launches, BTC/USDT pools will appear on NativeSwap and USDT/MOTO pools on Motoswap — representing the first Bitcoin-native stablecoin liquidity.

---

## Further Reading

- [docs.opnet.org](https://docs.opnet.org) — official token standard documentation
- [btc-vision/btc-runtime](https://github.com/btc-vision/btc-runtime) — `OP20` and `OP_NET` base classes
- [btc-vision/example-tokens](https://github.com/btc-vision/example-tokens) — reference implementations
