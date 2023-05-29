# 🤨 Metatransactions and Signature Replay

There are times when you want your dApp users to have a gas-less experience, or perhaps make a transaction without actually putting something on the chain. These types of transactions are called meta-transactions, and in this level, we will dive deep into how to design meta-transactions and also how they can be exploited if not designed carefully.

For those of you who have used OpenSea, ever noticed how OpenSea lets you make listings of your NFT for free? No matter what price you want to sell your NFT at, somehow it never charges gas beyond the initial NFT Approval transaction? The answer, Meta transactions.

Meta transactions are also commonly used for gas-less transaction experiences, for example asking the users to sign a message to claim an NFT, instead of paying gas to send a transaction to claim an NFT.

There are other use cases as well, for example letting users pay for gas fees with any token, or even fiat without needing to convert to crypto. Multisig wallets also only ask the last signer to pay gas for the transaction being made from the multisig wallet, and the other users just sign some messages.

```shell
gitpod /workspace/LW3-meta-transactions (main) $ npx hardhat test
Compiled 10 Solidity files successfully


  Lock
    Deployment
      ✔ Should set the right unlockTime (1133ms)
      ✔ Should set the right owner
      ✔ Should receive and store the funds to lock
      ✔ Should fail if the unlockTime is not in the future (39ms)
    Withdrawals
      Validations
        ✔ Should revert with the right error if called too soon
        ✔ Should revert with the right error if called from another account
        ✔ Shouldn't fail if the unlockTime has arrived and the owner calls it
      Events
        ✔ Should emit an event on withdrawals
      Transfers
        ✔ Should transfer the funds to the owner

  MetaTokenTransfer
    ✔ Should let user transfer tokens through a relayer with different nonces (232ms)
    ✔ Should not let signature replay happen (127ms)


  11 passing (2s)
  ```

# 🔓 Security Vulnerability

Can you guess what the problem is with the code we just wrote though? 🤔

Since the signature contains the information necessary, the relayer could keep sending the signature to the contract over and over, thereby continuously transferring tokens out of the sender's account into the recipient's account.

While this may not seem like a big deal in this specific example, what if this contract was responsible for dealing with money on mainnet? If the same signature could be reused over and over, the user would lose all their tokens!

Instead, the transaction should only be executed when the user explicitly provides a second signature (while staying within the rules of the smart contract, of course).

This attack is called Signature Replay - because, well you guessed it, you're replaying a signature.

# 🌟 Solving for Signature Replay
For even simpler contracts, you could resolve this by having some (nested) mappings in your contract. But there are 4 variables to keep track of here per transfer - sender, amount, recipient, and tokenContract. Creating a nested mapping this deep can be quite expensive in Solidity.

Also, that would be different for each 'kind' of a smart contract - as you're not always dealing with the same use case. A more general-purpose solution for this is to create a single mapping from the hash of the parameters to a boolean value, where true indicates that this meta-transaction has already been executed, and false indicates it hasn't.

Something like mapping(bytes32 => bool).

This also has a problem though. With the current set of parameters, if Alice sent 10 tokens to Bob, it would go through the first time, and the mapping would be updated to reflect that. However, what if Alice genuinely wants to send 10 more tokens to Bob a second time?

Since digital signatures are deterministic, i.e. the same input will give the same output for the same set of keys, that means Alice would never be able to send Bob 10 tokens again!

To avoid this, we introduce a fifth parameter, the nonce.

The nonce is just a random number value, and can be selected by the user, the contract, be randomly generated, it doesn't matter - as long as the user's signature includes that nonce. Since the exact same transaction but with a different nonce would produce a different signature, the above problem is solved!

# ⛓️ Sample Hardhat Project

This project demonstrates a basic Hardhat use case. It comes with a sample contract, a test for that contract, and a script that deploys that contract.
Try running some of the following tasks:

```shell
npx hardhat help
npx hardhat test
REPORT_GAS=true npx hardhat test
npx hardhat node
npx hardhat run scripts/deploy.js
```