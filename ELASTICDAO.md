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

### [H-03] Inﬁnite minting of tokens by exploiting eternal storage pattern on Token.sol

An attacker can change the expected DAO address to their own address in the `models/Token.sol` contract. The attacker can change the token parameters so that they receive much more ETH for their shares when exiting from the DAO. Attackers could also steal all funds from the DAO, effectively breaking ElasticDAO’s model.

### [H-04] Conﬁgurator contract allows inﬁnite minting of tokens

Even if a conﬁgurator address is hard-coded in the `Ecosystem.sol` contract, attackers can call the `buildToken` function on the `services/Configurator.sol` contract and change nearly all of the parameters to be attacker-controlled. The attacker bypasses the authorization check and gains write access to ﬁelds of the DAO ecosystem.

### [H-05] New users can be blocked from joining

The `join` function requires users to send an exact amount of ETH for the required shares. Attackers can survey the mempool for `join` transactions and send a tiny amount of wei to the DAO contract to change the required ETH value to a different value than the user submitted. This grieﬁng attack would prevent new users from joining the DAO.

Malicious actors might use this attack to prevent new votes from derailing the outcome of a proposal. Normal DAO usage might also block new users from joining. Token curve parameters change as a result of new join transactions. During times of high demand, many transactions will fail and effectively result in a self-DOS.

### [H-06] Incorrect event parameters in `transferFrom` function

The `emitApproval` event should occur when the `msg.sender` is not equal to `_from`. The event should be `emitApproval(from, msg.sender, _allowances[_from][msg.sender])`; instead of `emit Approval(msg.sender, _to, _allowances[_from [msg.sender]`). This error may negatively impact off-chain tools that are monitoring critical transfer events of the governance token.

### [H-07] Minter can call functions reserved for DAO addresses

In `ElasticGovernanceToken.sol`, the onlyDAO modiﬁer is meant to only allow a DAO address to call functions like `setBurner` and `setMinter`. However, as currently written, this modiﬁer allows the `msg.sender` to either be a DAO address or the minter address.
