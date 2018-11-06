---
eip: <to be assigned>
title: Earnings Standard
author: Roland Kofler (@rolandkofler), Dr. Sebastian Bürgel (@SCBuergel)
discussions-to: https://github.com/ethereum/EIPs/issues/1526
status: Draft
type: Standards Track
category: ERC
created: 2018-10-23
---

<!--You can leave these HTML comments in your merged EIP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new EIPs. Note that an EIP number will be assigned by an editor. When opening a pull request to submit your EIP, please use an abbreviated title in the filename, `eip-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EIP.-->
Tokens that generate returns for the token holder should have a standard way for processing payouts. Many actors can profit from the resulting interopability.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
This ERC represents a single standard for *earnings* disembursement and claiming. It aims to be compatible with (and even agnostic/ orthogonal to) fungible and non-fungible token standards like ERC20 and ERC777.
The term *earnings* and *returns* is used interchangably for anything that represents a surplus on an asset or service.

## Motivation
<!--The motivation is critical for EIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the EIP solves. EIP submissions without sufficient motivation may be rejected outright.-->
Security Tokens are emerging quickly as a new innovation in the blockchain realm. Standardization is a pressing issue (See ERC1400 et.al.). A small isolated topic is the distribution of *earnings* like *stock dividends*, *bond interests*, *staking revenue* etc.

A earnings standard...
1. ...is useful to 
    * crypto investment funds 
    * (de-)centralized exchanges
    * wallets 
    * staking pools
    * ...
2. ...is limited to *a well defined scope*, that can be expressed without touching other needed realms like regulations and taxation or bearing incompatibilities with existing standards like ERC20.
3. ...is *generic enough* to comprehend all variants of revenue generating financial instruments possible:
    * stock dividends
    * real estate investment trusts (REIT) returns
    * bond interests
    * staking revenues
    * intellectual property royalties 
    * asset rental income
    * ... and many more
    
Therefore we believe that there is an opportunity for an Ethereum Request for Comment (ERC) targeting earning disbursement, that can become a de facto standard in the spirit of the famous ERC20.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and [others](https://github.com/ethereum/wiki/wiki/Clients)).-->


### Earning interface

The generic interface regulates the disbursment and the reclaiming of *returns on assets*.
```
pragma solidity ^0.4.25;

interface Earning {

     /**
     * Looks if claiming earnings is currently possible for a claimer. 
     * For example the claimer might not have performed KYC and is not entitled to claim the returns,
     * or he does not own the asset he is claiming
     * @param _claimer the asset owner for the returns, can be `msg.sender`
     * @return _returnCode, normally `0` if everything went well.
     **/
    function canClaim(address _claimer)
    public
    returns (uint256 _returnCode);
    
    /**
     * Looks up if there are current earnings to be disbursed to the claimer
     * Gives back the allocated amounts even if `canClaim` signals objection to a payout
     * @param _claimer the asset owner for the returns, can be `msg.sender`
     * @return _tokens array of tokens for disbursement (independend if ERC20, ERC721 etc.)
     * @return _allocation the number of tokens disbursed
     **/
    function getEarningsOwed(address _claimer)
    public
    returns (address[] _tokens, uint256[] _allocation);
    
     /**
     * Looks up if there are current earnings of Eth to be disbursed to the claimer
     * Extra function for Ether because it has no address.
     * Gives back the allocated amounts even if `canClaim` signals objection to a payout
     * @param _claimer the asset owner for the returns, can be `msg.sender`
     * @return _tokens array of tokens for disbursement
     * @return _allocation the number of tokens disbursed
     **/
    function getEtherEarningsOwed(address _claimer)
    public
    returns (uint256 _allocation);

    /**
     * Claims earnings on assets. The owner must have the right to claim or the (see `canClaim()`).
     * Same for Ether and Tokens (independend if ERC20, ERC721 etc.)
     * @param _beneficiary the destination where the asset owner wants to transfer returns, can be `msg.sender`
     * @return _returnCode, normally `0` if everything went well.
     **/
    function claimEarningsFor(address _beneficiary)
    public
    returns (uint256 _returnCode);

    /**
     * Event for successfully claiming returns.
     * @param _claimer the asset owner for the returns, can be `msg.sender`
     * @param _beneficiary the destination for the returns, can be `msg.sender`
     **/
    event EarningsClaimed(
        address indexed _claimer,
        address indexed _beneficiary
    );

}
```
In the proposed interface are three functions.

### Optional Return on Claims Periods
TO BE Discussed: makes it sense to insert also a generic disbursment for the company?
```
    /**
     * Disburses earning to the smart contract and initializes the Returns
     * Distribution Period.
     * @dev marks implicitly the Declaration Date i.e. when claim disbursement
     *      is announced. OPTIONAL parameters can be `require`d to be `0`
     * @param _exDividendDate OPTIONAL when owning claims entitled to returns
     * @param _recordDate OPTIONAL when the returns on claims eligibility
     *        is recorded, normally the same date as _exDividendDate
     * @param _payoutDate MANDATORY when return on claims can be claimed
     * @param _clawBackDate OPTIONAL when unclaimed funds can be given to a
     *        beneficiary
     * @param _tokens array of tokens for disbursement, additionally Ether can
     *        be sent
     * @param _allocation the number of tokens disbursed, the last entry can be
     *        the amount of Ether
     * @return _returnCode, normally `0` if everything went well.
     **/
    function disburseEarnings (
        uint256 _exDividendDate,
        uint256 _recordDate,
        uint256 _payoutDate,
        uint256 _clawBackDate,
        address[] _tokens,
        uint256[] _allocation
    )
    payable
    public
    returns (uint256 _returnCode);
    
    
    /**
     * Event when a Earnings Distribution Period is started
     * @dev OPTIONAL parameters can be `0`
     * @param _exDividendDate OPTIONAL when owning claims entitled to returns
     * @param _recordDate OPTIONAL when the returns on claims eligibility
     *        is recorded, normally the same date as _exDividendDate
     * @param _payoutDate MANDATORY when return on claims can be reclaimed
     * @param _clawBackDate OPTIONAL when unclaimed funds can be given to a
     *        beneficiary
     **/
    event EarningsOnClaimPeriodStarted(
        uint256 _exDividendDate,
        uint256 _recordDate,
        uint256 _payoutDate,
        uint256 _clawBackDate
    );
 ```

| Type      | Date            | Desciption                                         |
| ----------|:---------------:| -------------------------------------------------- |
| Optional  |Declaration Date |	claim disbursement is announced                    |
| Optional	|Ex-Dividend Date |	until when owning claims entitled to returns       |
| Optional	|Record Date	  | when the returns on claims eligibility is recorded |
| Mandatory	|Payout Date	  | when return on claims can be reclaimed             |
| Optional  |Clawback Date	  | when unclaimed funds can be given to a beneficiary |

![Figure: the common sequence during returns distribution dates in time. Optional dates are dashed.](/assets/eip-dividend/20181710_DividendTokenStandard_ResearchingtheProblemDomain.png)
Figure: the common sequence during returns distribution dates in time. Optional dates are dashed.

### Optional building block for  internal snapshots cutoff dates
It is recommendable to integrate a standard to track the transfer of the
*claims* behind the dividend rights.
For example Validity Labs has proposed recently a [ERC20Snapshot.sol](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/617d5278865da56455fd64d149ff1f6ff6071f1d/contracts/token/ERC20/ERC20Snapshot.sol#L44)
 for the Open Zeppelin ecosystem.

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

A first [analysis of the problem domain](https://docs.google.com/document/d/1ERjxWZbsGXp4J6ZKotyoniTlLNjaZO4JSwg66UHc-p0/edit?usp=sharing) was made with expert interviews from the
domains of (1) investment industry and (2) law.

      Further inquiries and workshops with the Ethereum Security Token Comunity are currently in progress

## Backwards Compatibility
<!--All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

Currently we expect no backwards incompatibilities or incompatibilities with
other standards.

## Test Cases
<!--Test cases for an implementation are mandatory for EIPs that are affecting consensus changes. Other EIPs can choose to include links to test cases if applicable.-->

## Implementation
<!--The implementations must be completed before any EIP is given status "Final", but it need not be completed before the EIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
