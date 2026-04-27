# AMM / DEX Playbook

- Sandwich attack on swaps (front-run + back-run)
- Price manipulation via flash loans within same tx
- Slippage protection: is `minAmountOut` enforced? Can it be set to 0?
- Fee-on-transfer token compatibility
- Reentrancy via token callbacks (ERC777, ERC1155)
