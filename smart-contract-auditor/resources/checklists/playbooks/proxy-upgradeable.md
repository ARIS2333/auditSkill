# Proxy / Upgradeable Playbook

- Storage collision between proxy and implementation
- Uninitialized implementation (can attacker call `initialize`?)
- Function selector collision between proxy and implementation
- Storage gaps (`__gap`) — are they properly sized?
- `delegatecall` context: `msg.sender` and `msg.value` are preserved, storage is proxy's
