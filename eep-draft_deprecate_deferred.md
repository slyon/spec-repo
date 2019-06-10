---
eep: Deprecate Deferred Transactions
title: <EEP title>
author: <a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s), e.g. (use with the parentheses or triangular brackets): FirstName LastName (@GitHubUsername), FirstName LastName <foo@bar.com>, FirstName (@GitHubUsername) and GitHubUsername (@GitHubUsername)>
discussions-to: <URL>
status: Draft
type: <Standards Track (Core, Networking, Interface)  | RFC | Meta>
category (*only required for Standard Track): <Core | Networking | Interface>
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
requires (*optional): <EEP number(s)>
replaces (*optional): <EEP number(s)>
---

<!--You can leave these HTML comments in your merged EEP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new EEPs. Note that an EEP number will be assigned by an editor. When opening a pull request to submit your EEP, please use an abbreviated title in the filename, `eep-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EEP.-->

Deferred transactions are now deprecated.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

Deferred transactions are now deprecated. Likewise, wait weights in account
authorities are also deprecated. This EEP explains some of the motivation
behind this, and presents some potential replacements for existing use cases.

## Motivation
<!--The motivation is critical for EEPs that want to change the EOSIO protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the eep solves. EEP submissions without sufficient motivation may be rejected outright.-->

Deferred transactions have been a major source of bugs and have hindered potential
enhancements. Here are some of the issues:

* They complicate nodeos. Deferred transactions have a non-obvious set of rules which define their
  lifespan. They complicate consensus because of the large number of ways they could fail,
  compared to normal transactions. Contract-generated deferred transactions complicate this even
  further with their onerror notifications. Deferred transactions have many exemptions to the normal
  transaction flow through nodeos.
* They complicate history. Non-deferred transactions have a simple rule: if they appear in a block
  in the current fork, then they successfully executed in that fork. Deferred transactions
  are more complicated; they can have one of 5 different status values. They can appear in two
  blocks with 2 different status values (delayed followed by something else). They can be paired
  with an error-handling transaction with its own status. The block, if any, which holds the 
  deferred transaction's content isn't normally the block that contains the final
  status. These complications have led to bugs in plugins and in external tools which deal with
  history. In some cases, attackers have exploited app confusion over deferred transaction
  status.
* Contract authors often assume that contract-generated deferred transactions will execute. In
  many cases, they don't. There is no way to guarantee these will execute without sacrificing
  the safety of the chain.
* Contract authors often assume that their onerror handlers will execute when a
  contract-generated deferred transaction fails. In many cases, they don't. Just like the
  deferred transactions themselves, there is no way to guarantee the onerror handlers will
  execute without sacrificing the safety of the chain.
* They complicate account sales and account recovery. Once someone has control over an account's
  owner, they can create a deferred transaction which revokes ownership back to them at a later
  date. This can execute after they sell the account to someone else, or, if the account was
  stolen, the account was restored to the legitimate owner.

## Alternatives

* `eosio.msig` currently uses deferred transactions. It could switch to inline actions instead.
   This has some advantages:
  * If `exec` fails, the status of the failure will be in the receipt of the transaction
    which used `exec`, simplifying diagnosis.
  * If `exec` fails, it can be retried after fixing the problem. e.g. by increasing resources.
* `eosio.wrap` also uses deferred transactions. It could switch to inline actions instead.
* Some users use wait weights in combination with deferred transactions to protect their accounts.
  They could switch to contract-based protection instead. Contracts which provide protection
  services could use the [Contract Authentication](eep-draft_contract_trx_auth.md) and
  [Forwarding Authorizations](eep-draft_contract_fwd_auth.md) proposals to implement their
  own authorization requirements.
* Contracts sometimes use deferred transactions to resume long-running calculations, to do
  regularly-scheduled maintenance tasks, or to add a delay to an action. These contracts
  already need a backup mechanism since deferred transactions are unreliable. e.g. 
  eosio.system allows users to use `refund` action if a deferred transaction fails.
  Contracts' backup solutions could become their primary solutions.

The `eosio.msig` and `eosio.wrap` changes require increasing the `max_inline_action_size`
and `max_inline_action_depth` consensus parameters. These need to be large enough to allow
`setcode` and `setabi` inline actions.

## Copyright