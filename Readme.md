# TokenApp Smart Contract
The testing framework is truffle and the goal is to test 
as many every corner case as possible in the smart contract.
The contract is based on ERC20, and uses code from the ERC20 Token
Standard [page](https://theethereum.wiki/w/index.php/ERC20_Token_Standard).

Although a partial ECR20 token without the functions `approve(...)`, 
`allowance(..)`, `transferFrom(...)`, and the event `Approval(...)` fits the purpuse, 
it was decided to implement the full standard. The token name is Modum Token
and the symbol MOD (not to be confused with 
[the file format](https://en.wikipedia.org/wiki/MOD_(file_format))).

The maximum amount of tokens are 30mio, where 9.9mio tokens are locked and can be only 
unlocked by the token holders by voting. The modum company itself initially 
does not have any tokens. 

This contract has 2 additions:

* **Bonus payment (airdrop)** A bonus payment starts by calling the default function. There 
 is no restriction, thus, anyone can pay ethers to this contract. In order to avoid
 rounding issues, only a multiple wei of the amount of unlocked tokens is considered.
 The modulo of the payin and unlocked tokens (remainder) is kept and added for the next
 payout. Thus, e.g., if there are 15'000'000 unlocked tokens and the bonus is 1 ETH 
 (1'000'000'000'000'000'000 wei), then the remainder is 10'000'000 and decucted from the
 bonus. Thus, the bonus payment is 999'999'999'990'000'000 / 15'000'000 = 66'666'666'666 
 wei per token. For a next payment e.g., 0.5 ETH (500'000'000'000'000'000 wei), the remainder 
 is added to 500'000'000'010'000'000. If any previous bonus payment has been done, the current
 wei per token is added to the previous one.

* **Voting** A vote starts by the owner of the contract calling 
`proposal(string _addr, bytes32 _hash, uint _value)`. The `addr` stores where the proposal
 document can be found, e.g. https://example.com/voting-document.pdf and the `hash` is the 
 SHA3 checksum of the document. The `value` in this proposal are the number of modum tokens
 that will we moved from the locked funds to the modum account. 
 
 The tokens are only moved from the locked funds to the modum account if a majority 
 approves. Every token holder can vote with `vote(bool _vote)` and 1 modum token is 
 equal to 1 vote.
 
 Once the voting phase is over (2 weeks), the owner can call `claimProposal()`. It was 
 decided against counting in blocks, as the current blocktime is 
 [>20sec](https://etherscan.io/chart/blocktime) and increasing due to the 
 [ice age](https://www.cryptocompare.com/coins/guides/what-is-the-ethereum-ice-age/), 
 thus, not suitable for defining a time period.
 
 In any case (accepted or rejected vote), after the voting phase, the owner has to 
 call `claimProposal()` in order to create new proposals. After the call, the proposal
 state is reset.
  
 
Since the voting and bonus payment are based on the number of tokens, it is important to 
handle the case of increasing / decreasing tokens in order not to claim more bonus or
voting rights...

Minting is done once, and bonus is calculated off-contract...