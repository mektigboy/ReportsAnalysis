# ELASTICDAO

Only high and medium severity findings.

## High Severity

### [H-01] Inﬁnite minting of tokens by exploiting eternal storage pattern on DAO.sol

Attackers can overwrite metadata in the models/DAO.sol eternal storage contract by using the `serialize` call to change the expected configurator address to the attacker’s address.

This allows the attacker to change the DAO data and potentially mint infinite tokens for themselves. We consider this risk high severity because it would disrupt the economics of the DAO in a manner that would prevent the system from performing.
