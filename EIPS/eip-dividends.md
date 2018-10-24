---
eip: <to be assigned>
title: Dividend Standard
author: Roland Kofler (@rolandkofler), Dr. Sebastian Bürgel @SCBuergel
discussions-to: https://github.com/ethereum/EIPs/issues/1526
status: Draft
type: Standards Track
category ERC
created: 2018-10-23
---

<!--You can leave these HTML comments in your merged EIP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new EIPs. Note that an EIP number will be assigned by an editor. When opening a pull request to submit your EIP, please use an abbreviated title in the filename, `eip-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EIP.-->
Tokens that generate returns for the holder should implement a standard way for processing payouts.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
Represents a single standard for dividend disembursement and dividend claiming, and to "returns on claims" in general. It aims to be compatible with (and even agnostic/ orthogonal to) fungible and non-fungible token standards like ERC20 and ERC777.

## Motivation
<!--The motivation is critical for EIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the EIP solves. EIP submissions without sufficient motivation may be rejected outright.-->
Security Tokens are emerging quickly as a new innovation in the blockchain realm. Standardization is a pressing issue (See ERC1400 et.al.). A small isolated topic is the distribution of dividends.

But dividends are only one of many forms of *returns on asset claims*, the standard thus tries to be as generic as possible, being applicable e.g. for interests on bonds.
**Nota Bene:** Because of this generalization, in this document (1) the more generic term *returns on claims* is interchangeable with (2) the more popular term *dividends*.
A dividend standard (more general: a "return on claim standard"):
1. Is useful to *crypto investment funds, (de-)centralized exchanges, wallets, security token users, tax authorities* and *token creators*.
2. Is limited in *a well defined scope*, that can be expressed without touching other needed realms like regulations and taxation or bearing incompatibilities with existing standards like ERC20.
3. Is *generic enough* to comprehend all variants of revenue generating financial instruments possible (not only stock dividends, but REITs, bond interests etc.)
Therefore we believe that there is an opportunity for an Ethereum Request for Comment (ERC) targeting dividend disbursement, that can become a de facto standard in the spirit of the famous ERC20.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and [others](https://github.com/ethereum/wiki/wiki/Clients)).-->


### ReturnOnClaim interface

The interface regulates the disbursment and the reclaiming of *returns on claims*.
```
pragma solidity ^0.4.25;

interface ReturnOnClaim {

    /**
     * Disburses returns to the smart contract and initializes the a Returns
     * Distribution Period.
     * @dev marks implicitly the Declaration Date i.e. when claim disbursement
     *      is announced. OPTIONAL parameters can be `require`d to be `0`
     * @param _exDividendDate OPTIONAL when owning claims entitled to returns
     * @param _recordDate OPTIONAL when the returns on claims eligibility
     *        is recorded, normally the same date as _exDividendDate
     * @param _payoutDate MANDATORY when return on claims can be reclaimed
     * @param _clawBackDate OPTIONAL when unclaimed funds can be given to a
     *        beneficiary
     * @param _tokens array of tokens for disbursement, additionally Ether can
     *        be send
     * @param _allocation the number of tokens disbursed, the last entry can be
     *        the amount of Ether
     * @return _returnCode, normally `0` if everything went well.
     **/
    function disburseReturns (
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
     * Claims returns on assets
     * @param _beneficiary the destination for the returns, can be `msg.sender`
     * @return _returnCode, normally `0` if everything went well.
     **/
    function claimReturnsFor(address _beneficiary)
    public
    returns (uint256 _returnCode);

    /**
     * Looks if reclaiming returns is currently possible
     * @param _claimer the destination for the returns, can be `msg.sender`
     * @param _beneficiary the destination for the returns, can be `msg.sender`
     * @return _returnCode, normally `0` if everything went well.
     **/
    function canClaim(address _claimer, address _beneficiary)
    public
    returns (uint256 _returnCode);

    /**
     * Claims returns on assets
     * @param _beneficiary the destination for the returns, can be `msg.sender`
     * @return returnCode, normally `0` if everything went well.
     **/
    function getReturnsOwed(address _beneficiary)
    public
    returns (uint256 _returnCode);

    /**
     * Event when a Returns Distribution Period is started
     * @dev OPTIONAL parameters can be `0`
     * @param _exDividendDate OPTIONAL when owning claims entitled to returns
     * @param _recordDate OPTIONAL when the returns on claims eligibility
     *        is recorded, normally the same date as _exDividendDate
     * @param _payoutDate MANDATORY when return on claims can be reclaimed
     * @param _clawBackDate OPTIONAL when unclaimed funds can be given to a
     *        beneficiary
     **/
    event ReturnOnClaimPeriodStarted(
        uint256 _exDividendDate,
        uint256 _recordDate,
        uint256 _payoutDate,
        uint256 _clawBackDate
    );

    /**
     * Event for successfully claiming returns.
     * @param _claimer the destination for the returns, can be `msg.sender`
     * @param _beneficiary the destination for the returns, can be `msg.sender`
     **/
    event ReturnsClaimed(
        address indexed _claimer,
        address indexed _beneficiary
    );

}
```
In the proposed interface are three functions.

### Timings of a Return on Claims Period
![](../../assets/eip-dividend/20181710_DividendTokenStandard_ResearchingtheProblemDomain.png)

Figure: the common sequence during returns distribution dates in time. Optional dates are dashed.

### Optional building block for  internal snapshots cutoff dates
It is recommendable to integrate a standard to track the transfer of the
*claims* behind the dividend rights.
For example Validity Labs has proposed recently a [ERC20Snapshot.sol](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/617d5278865da56455fd64d149ff1f6ff6071f1d/contracts/token/ERC20/ERC20Snapshot.sol#L44)
 for the Open Zeppelin ecosystem

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
