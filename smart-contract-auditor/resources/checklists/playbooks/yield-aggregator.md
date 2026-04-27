# Yield Aggregators / Vaults (Multi-Strategy) Playbook

- Strategy composability — can a malicious or compromised strategy drain the vault?
- Harvest timing manipulation — front-running harvest to capture yield without proportional time exposure
- Loss socialization — when one strategy loses, is the loss spread fairly across all depositors?
- Strategy migration — are assets safe during the transition between old and new strategies?
- Reward token re-investment slippage — can sandwich attacks extract value during auto-compound?
- Emergency withdrawal paths — do they work when the underlying protocol is paused or compromised?
