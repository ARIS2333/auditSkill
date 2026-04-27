# Staking / Restaking Playbook

- Withdrawal delay manipulation (can attacker front-run slashing by initiating withdrawal?)
- Slashing edge cases — does share accounting remain correct after partial slashing? Can slashing create bad debt?
- Operator trust boundaries — what can a malicious operator do? Can they grief delegators?
- Reward distribution timing — can staking just before reward distribution and unstaking after capture disproportionate rewards?
- Exchange rate manipulation between staking token and underlying
- Re-delegation during pending withdrawal — can user double-count stake?
- Minimum stake / dust stake DoS (many tiny stakes to bloat operator sets)
