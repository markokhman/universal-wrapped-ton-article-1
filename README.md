Universal WTON initiative and how we make TON better together
===

Hello guys, it's been more then a month since I've got involved in the Wrapped TON initiative and I'm sharing with you some overview as well as my thoughts on it. 

If you are familiar with FunC programming or aim at starting on this path - this will be very interesting for you from both technical and community sides. To follow up with the latest news on Universal WTON, please have a look at this TEP: 
https://github.com/ton-blockchain/TEPs/pull/102

I would like to address this article to newcomers in TON as well as experienced developers, so here is the agenda:
1. What is WTON and why to even have it?
2. How do community initiatives work in TON?
3. Current suggested Universal WTON standard
4. Fresh new input from TON Core team on the WTON
5. Please take part in the discussion

Are you ready? Let's jump into it! 

# 1. What is WTON and why to even have it?

Interaction with different kinds of assets requires additional conditional logic. To avoid that and unify interaction between TON and jettons of TEP-74 and TEP-89 standards, we introduce a concept of **WTON - a jetton that is locking TON 1-to-1 on mint and releases TON on burn.**

Implementation of a wrapped TON contract doesnt require a lot of effort. As a matter of fact, several implementations already exist. The problem is that numerous different wrapped TONs (wTON, jTON etc) created by different developers bare lots of risks:

- **Financial risks:** there is no guarantee that a particular WTON doesn't hold security vulnerabilities left deliberately or by mistake. Getting security audits and certifications of the same quality for all the many WTONs is unrealistic.
- **Ecosystem fragmentation and no single API:** this is a real problem for developers who are building products that will have to use a variety of WTONs. Imagine a wallet developer who wants to correctly display the total amount of WTON, including regular and all kinds of wrapped ones.
- **No parity of features:** each WTON will have its own unique set of features, which will lead to additional issues with support.


# 2. How do such initiatives work in TON?

While working with Universal WTON I got familiar with a number of great tools, that enable any builder to make impact. Let me explain.

**TEPs (TON Enhancement Proposals).** 

The main goal of TON Enhancement Proposals is to provide a convenient and formal way to propose changes to TON Blockchain and standardize ways of interaction between different parts of ecosystem. Proposal management is done using GitHub pull requests, the process is described formally in [TEP-1](./text/0001-tep-lifecycle.md).

Here is a simple list of steps to use the TEP instrument:
1. Discuss your proposal with community first, for example in TON Dev chat ([en](https://t.me/tondev_eng)/[ru](https://t.me/tondev)).
2. Read [TEP-1](./text/0001-tep-lifecycle.md) to understand proposal management process and follow the guideline.
5. Submit a pull request.

**The TON Footsteps**

TON Footsteps is another cool instrument that can help you to fund your initiative. It's run similarly to TEPs. You can go the official [TON Footsteps repository](https://github.com/ton-society/ton-footsteps), propose your initiative according to guidlines and let the community support you on this path. 

Based on the analysis of your initiative and the level of support from the community, the TON Footsteps committee will decide wheather to accept and fund it. I find it very cool, that things like that enable anybody to take part in the course path of TON and improve it.

So, to sum up - The Universal WTON initiative was funded by **TON Footsteps** and is run through official **TON Enhancement Proposals** respository.

# 3. Current suggested Universal WTON standard
As the first step of TEPs usages states, we've discussed the WTON proposal with the community. With help of guys from [TON Foundation](https://ton.org/) and [FS Labs](https://fslabs.io/) we've gathered representatives from all major existing DEXs (Decentralized Exchange) developers and started a solid discussion on how to implement WTON in the most effecient way. Following is what we came up with.
> You should definitely consider looking into [the actual TEP Pull Request](https://github.com/ton-blockchain/TEPs/pull/102), as it has much more detailed explanation along with specifications, solution drawbacks and alternatives.

As mentioned above, the universal WTON has a single source of origin, a deployed minter contract that has no owner/admin functinalities.

Two core functionalities of WTON are **wrapping** and **unwrapping** TON.

### Wrapping

In order to wrap WTON, send a `mint` operation code with a message. WTON minter will reserve `amount + minimal_balance()` TON on the minter contract and will send the rest to a WTON wallet of `recipient` within `internal_transfer` message.

One of the most important things is the ability to attach `forward_amount` and `forward_payload` to build a pipeline of transactions.

The message should be rejected if:

- `msg_value` is less than `amount + forward_amount + gas_fee`
- `recipient` is in different workchain than the minter

### Unwrapping

In order to unwrap WTON jettons back to TON, `burn` operation code should be used.

Our deployed implementation relies on existing operation `burn` (including `custom_payload`) but uses a new operation `burn_notification_ext` instead of `burn_notification`.
The only reason of replacing `burn_notification` by a new `burn_notification_ext` is inability to attach a `custom_payload`.

After burning, a jetton wallet sends `burn_notification_ext` to a minter. As soon as WTON minter receives `burn_notification_ext` message, it decreases `total_supply` by `amount`,
reserves `total_supply + minimal_balance()` and sends a message with `release` operation code to `response_destination` attaching the rest of TON and optional `custom_payload`.

The message with `burn` operation should be rejected if:

- `response_destination` is not specified
- `amount` is less than jetton balance of jetton wallet
- `msg_sender` is not an owner of jetton wallet
- `msg_value` is not enough for handling both messages `burn` and `burn_notification_ext`


# 4. Fresh new input from TON Core team on the WTON
Here we expereince the power of the TEP tool, as all the proposals there are regularly reviewed by core team and so was our proposal.

After some waiting time, we've received a fresh new idea from Ston.fi developer [Dario](https://github.com/dariotarantini) and [EmelyanenkoK](https://github.com/EmelyanenkoK) - this one of the Core team developers as we can see on the commits to the main TON monorepo. [EmelyanenkoK](https://github.com/EmelyanenkoK) also [implemented a draft solution](https://github.com/EmelyanenkoK/wTON), that can help you better understand the overall idea. 

> I will briefly explain the new inputs here, but you should definitely go into the [the actual TEP Pull Request](https://github.com/ton-blockchain/TEPs/pull/102) and follow along with the discussion.

To sum up the idea, i will quote the [EmelyanenkoK's message on the PR](https://github.com/ton-blockchain/TEPs/pull/102#issuecomment-1378829623):
> I want to emphasize some aspects of @dariotarantini proposal:
1.each wTON wallet works as minter/burner itself - no need for additional interaction with minter which means one or even two transaction shorter paths
2.we can reformulate this scheme in the following terms: in original proposal when we want to send TONs we actually send IOU jettons which is redeemable as TON, but in scheme proposed by Dario we send TON themselves but disguised as jetton transfer for compatibility.

This example showcases, that some great ideas can be figured out while collaborating. 

# 5. Please take part in the discussion
So now, after getting such solid input from both Core team and community member, we are going to proceed with discussions on the details of how this should be applied on Universal WTON.

I want to invite you to participate in this discussion. IMHO, this is the best way to better understand TON, it's community and it's brightest future.

Follow the [link](https://github.com/ton-blockchain/TEPs/pull/102) and let us know what you think!

---


Written by [Mark Okhman](https://github.com/markokhman)
To stay in touch, please follow my [Telegram](https://t.me/markokhmandev) and [YouTube](https://www.youtube.com/@markokhman?sub_confirmation=1) channels.
