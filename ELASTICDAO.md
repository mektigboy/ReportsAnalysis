# ELASTICDAO

Only high and medium severity findings.

## High Severity

### [H-01] Inﬁnite minting of tokens by exploiting eternal storage pattern on DAO.sol

Attackers can overwrite metadata in the `models/DAO.sol` eternal storage contract by using the `serialize` call to change the expected conﬁgurator address to the attacker’s address.

This allows the attacker to change the DAO data and potentially mint inﬁnite tokens for themselves. We consider this risk high severity because it would disrupt the economics of the DAO in a manner that would prevent the system from performing.

### [H-02] Inﬁnite minting of tokens by exploiting eternal storage pattern on Ecosystem.sol

A similar exploit is possible because of the `models/Ecosystem.sol` eternal storage contract. Attackers can change the expected address to their own address while bypassing the authorization check for this function. This allows the attacker to change the DAO ecosystem data.

Gaining full access to the DAO ecosystem data is also possible by changing the `daoModelAddress` ﬁeld to an attacker-controlled proxy contract. This attack vector allows
attackers to mint inﬁnite tokens for themselves and potentially break all DAOs created by the ElasticDAO system.
