---
description: >-
  The politically selected governance body responsible for managing the proposal
  system for the benefit of voters.
---

# Council

## Introduction

The council is a fixed size committee, up for election at regular intervals by token holders, tasked with the role of voting on proposals in the proposal system.

## Roles

The relevant roles in the council system are

* **Voters:** Anyone who stakes for the purposes of influencing the outcome of an election. Is not tied to membership, so a member can vote multiple times for different candidates from different accounts.
* **Candidate:** A member who has staked and stands as an alternative for councilorship in an ongoing election cycle.
* **Councilor:** A member who has stood as a candidate in an election and won a place in a council. Has the primary responsibility to participate in voting on and deliberating around proposals in the proposal system.

## Concepts

### Staking

There are two kinds of staking associated with the council: **voting and council membership candidacy.** Importantly, one of the distinct characteristics of this staking from everything else i the runtime \(see [Staking](../key-concepts/staking.md)\) is that it allows the redeployment of already staked funds in the system for other purposes. This means, for example, that a validator or nominator can vote using some part of that stake. In this same spirit, it is also possible for a council candidate to stake in an election which is already locked up for staking due to being part of the currently active council. Voting periods are non-overlapping, so no reuse considerations are even required.

The staking is implemented by two set of locks: one voting lock and two council candidacy locks. The two council candidacy locks are meant for odd and even numbered election cycles. Having these distinct locks for adjacent council periods, in combination with the non-stacking behavior of locks, gives a simple way to implement the intended reuse.

### Budget

The budget of the council is the root resource pool for all token minting on the platform, _with the exception of validator rewards_. The lifetime of the budget is divided into periods, called _budget periods_, which occurs every `BUDGET_PERIOD_LENGTH` blocks. The number of tokens added to the budget at the end of each such period is held in a mutable parameter denoted as `budget_increment` .

Whenever one of the following actions occur, the budget is impacted as described.

| Event | Budget Impact |
| :--- | :--- |
| Working group budget increased by `X > 0` | `-X` |
| Working group budget decreased by `X > 0` | `+X` |
| Spending proposal with amount `X > 0` | `-X` |
| Council reward payout `X > 0` | `-X` |
| Budget period ends | `+budget_increment` |

Events that negatively impact the budget balance can only occur if the impact does not occur if they exceed the balance of the budget.

_Notice that working group spending, such as lead spending, or subsystem specific spending, such as minting initial credit for invited new members, does not directly count against the council budget, but against the relevant working group budget._

### **Council**

The council has a fixed number of seats `NUMBER_OF_COUNCIL_SEATS` occupied by members, called _councilors_. The seats are always occupied, allowing the platform to dispose of all proposals they may come in at any time. The council body has two high level states described as follows.

* **Normal:** During this stage the council operates normally. After `NORMAL_PERIOD_LENGTH` blocks have passed since this period started, a transition is made to the election stage.
* **Election:** During this stage, not only does the council operate, but there is an election ongoing. Read more about elections the [Council](council.md#election) section below.

During both these stages, the following can or does occur.

#### Recover Failed Candidacy Stake

If someone announced their candidacy in an election, but did not end up winning, then they can at any time after the conclusion of that election cycle free their stake

#### Recover Voting Stake

If someone voted for a candidate in an election, they will and can free their stake at a later time. Importantly, a vote which was devoted to a losing candidate can be freed the moment the election cycle is over, while a vote which was devoted to a winner can only be freed after the announcing period of the next election begins. **The idea behind this asymmetry is to lock ....**

#### Rewards

Every `REWARD_PERIOD_LENGTH` blocks all councilors are paid out the same flat reward rate and any possibly outstanding owed reward. This rate is held in a mutable parameter, called the _councilor reward_, denoted as `councilor_reward` . During this payout, where councilors are processed in some consistent order, the crediting only occurs while the budget constraint is respected. For each payout, the constraint is tightened. If a councilors cannot be paid out in full, then the difference is added to their owed reward. When a council period ends, any owed reward and outstanding reward from the last payout, are attempted paid out, however if the budget does not allow it, then the councilor suffers the loss.

### Candidacy

A candidacy is defined by the following information

* **Member:** The member behind the candidacy.
* **Program:** A human readable description of the candidacy. Some socially enforced schema for the encoding of the program.
* **Staking account:** The account holding the stake for the candidate. After announcing the staking account will have locked up `REQUIRED_CANDIDACY_STAKE` under the relevant council lock. If the candidacy fails - either because the election cycle fails or the candidate receives too few votes, then this lock can be removed by the candidate, otherwise it remains on into the councilorship.

Note that, while there is no explicit identifier, a candidacy can be implicitly identified by a combination of the member, the order of this announcement for this member - as one could in principle announce and withdraw multiple times, and finally the election cycle number.

### Councilor

A councilor is defined by the following information

* **Member:** The membership to which this role corresponds.
* **Role account**: The account currently used to authenticate as this role.
* **Reward account**: The destination account to which periodic rewards are paid out.
* **Staking account:** Holds the stake currently associated with the role. Locks`REQUIRED_CANDIDACY_STAKE` under the relevant council lock which is recoverable when councilorship ends.
* **Owed reward:** The total reward this councilor was not paid over a number of payout periods where there was not sufficient funds in the council budget.
* **Ending statement:** An optional mutable human readable statement a councilor can provide which reflects their view on the council period. Can be updated multiple times during councilorship.

Notice that, while there is no explicit identifier, a councilorship can be implicitly identified by a combination of the member and the election cycle number.

### Vote

A vote is a defined by the following

* **Staking account:** Holds the stake associated with the vote. A given account can only be involved in a single vote for a given election cycle \(see [Council](council.md#election)\).
* **Staking balance:** The amount of funds in the staking account encumbered for this vote, will be no less than `MINIUMUM_VOTING_STAKE`.
* **Cycle Id:** The election cycle in which the vote was cast.
* **Stage:** The vote has two stages, being _sealed_ and _unsealed, each having the following associated information_
  * **Sealed:** This is the initial stage when a vote is submitted during a the voting period of an election. The only information available is called a voting commitment, which is a **opaque hash digest**.
  * **Unsealed:** This is the stage which occurs if the voter chooses to reveal the their sealed vote during the revealing stage. This has stage has information about a **valid candidate**, and a **nonce**, which when concatenated together are the pre-image of the initial hash digest.

Notice that, while there is no explicit identifier, a vote can be implicitly identified by a combination of the staking account and the election cycle number.

Unlocking the voting lock on the staking account requires an active recovery action on the voter, and it follows the following rules

* If the vote is for an ongoing election, then it is not recoverable.
* If the vote is for the last concluded election, then it is recoverable only if it was unsealed in favor of a losing candidate, otherwise it is not.
* If the vote is for any election before the last concluded, the it is always recoverable.

### Election

An election is the periodic process by which a new council is selected by voters among candidates running for a seat on the next council. Elections occur periodically, and each one has a sequence of stages referred to as the election cycle. Each cycle is identified with an id, called the _election cycle id,_ which is just the cycle number. An election will begin while the current council is active, and the sitting council is only relieved once a new one has been successfully elected. As will become clear, this process can go on for a unknown amount of time. The election cycle has the following stages

* **Announcing Period:** This is the first stage in the election cycle. During this time members can announce that they will stand as candidates for the next council. Such an announcement can later be withdrawn within this same period, without consequences. The same member can only have a single candidacy active at any given time, but can in principle announce and withdraw an unlimited number of times. Importantly, if less than the minimum number of candidates have announced by the end of this period, a new election cycle starts. All candidates can recover their stake from such a failed cycle instantly, but it requires action, and anyone wanting to stand for the next election will need to announce again.
* **Voting Period:** This is the stage where voters can submit votes in favor of candidates. The votes are sealed, meaning that it is only known that some account voted for an unknown, possibly invalid candidate, with a known amount of tokens.
* **Revealing Period:** During this stage, voters can reveal their sealed votes. Any valid vote which is unsealed is counter, and in the end a winning set of candidates is selected. Importantly, even if there is an insufficient number of valid votes revealed to render a set of winners with non-zero backing stake, the runtime will just pick a winning set deterministically. Importantly, if less than `NUMBER_OF_COUNCIL_SEATS` candidates receive at least one vote by the end of this period, a new election cycle starts. All candidates can recover their stake from such a failed cycle instantly, but it requires action, and anyone wanting to stand for the next election will need to announce again.

The stages and transitions, are summarized in the figure below.

![Election life-cycle stages.](../.gitbook/assets/election-1.png)

## Constants

The following constants are hard coded into the system, they can only be updated with a runtime upgrade.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>NUMBER_OF_COUNCIL_SEATS</code>
      </td>
      <td style="text-align:left">The number of council seats.</td>
      <td style="text-align:left"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>IDLE_PERIOD_LENGTH</code>
      </td>
      <td style="text-align:left">The number of blocks in the normal period.</td>
      <td style="text-align:left"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>ANNOUNCING_PERIOD_LENGTH</code>
      </td>
      <td style="text-align:left">The number of blocks in the announcing period</td>
      <td style="text-align:left"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>VOTING_PERIOD_LENGTH</code>
      </td>
      <td style="text-align:left">The number of blocks in the voting period.</td>
      <td style="text-align:left"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>REVEALING_PERIOD_LENGTH</code>
      </td>
      <td style="text-align:left">The number of blocks in the revealing period.</td>
      <td style="text-align:left"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>REWARD_PERIOD_LENGTH</code>
      </td>
      <td style="text-align:left">The number or blocks between each reward
        <br />payout to councilors.</td>
      <td style="text-align:left"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>BUDGET_PERIOD_LENGTH</code>
      </td>
      <td style="text-align:left">
        <p>The number of blocks between each time the the</p>
        <p>council budget is topped up.</p>
      </td>
      <td style="text-align:left"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>REQUIRED_CANDIDACY_STAKE</code>
      </td>
      <td style="text-align:left">The required amount of stake for a candidate.</td>
      <td style="text-align:left"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MINIMUM_VOTING_STAKE</code>
      </td>
      <td style="text-align:left">The minimum allowable stake in a vote.</td>
      <td style="text-align:left"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MAX_SALT_LENGTH</code>
      </td>
      <td style="text-align:left">The maximum length of salt is used to calculate a vote&apos;s sealed commitment.</td>
      <td
      style="text-align:left"><code>fill-in</code>
        </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MINIMUM_CANDIDATES_COUNT</code>
      </td>
      <td style="text-align:left">The minimum number of candidates needed for the election to become legitimate.</td>
      <td
      style="text-align:left"><code>fill-in</code>
        </td>
    </tr>
  </tbody>
</table>

## Extrinsics

### Announce Candidacy

#### Parameters

| Name | Description |
| :--- | :--- |
| `membership_id` | Membership id uniquely identifying the user. |
| `staking_account_id` | Staking account. |
| `reward_account_id` | Account receiving councilors rewards in the case candidate gets elected. |
| `stake` | Amount of currency user wants to stake for the candidacy. |

#### Conditions

* Candidacy Announcement phase is running.
* Candidacy is not announced repeatedly.
* `staking_account_id` has enough balance to be locked as candidacy stake.
* `staking_account_id` and `reward_account_id` are both associated with the member via `membership_id`
* The `stake` must be at least `REQUIRED_CANDIDACY_STAKE`.

#### Effect

* The user is added to the list of candidates for the current election and candidacy stake is locked for the duration of election.

### Withdraw Candidacy

#### Parameters

| Name | Description |
| :--- | :--- |
| `membership_id` | Membership id uniquely identifying the user. |

#### Conditions

* The Candidacy Announcement phase is running.
* The user's candidacy is currently announced.

#### Effect

* The user is removed from the list of candidates for the current election and candidacy stake is unlocked.

### Submit Sealed Vote

#### Parameters

| Name | Description |
| :--- | :--- |
| `staking_account_id` | Staking account. Derived from the Origin. |
| `commitment` | The sealed vote representation. |
| `stake` | Amount of currency user wants to stake for the vote. |

#### Conditions

* The Voting phase is running.
* The `staking_account_id` hasn't been used for voting in the current election yet. \(If you want to vote for multiple candidates, repeat vote with a different staking account\(s\).\)
* The `staking_account_id` has enough balance to be locked as a voting stake.
* The `stake` must be at least `MINIUMUM_VOTING_STAKE`.

#### Effect

* The sealed commitment is remembered, and the voting stake is locked until this election ends.

### Reveal Vote

#### Parameters

| Name | Description |
| :--- | :--- |
| `staking_account_id` | Staking account. |
| `salt` | The \(cryptographic\) salt used to calculate the sealed commitment. |
| `vote_option_id` | The user id of the candidate you voted for. |

#### Conditions

* The Revealing phase is running.
* The vote's sealed commitment has been previously cast via [Submit Sealed Vote](council.md#user-content-submit-sealed-vote) using `staking_account_id` in the current election.
* The sealed commitment cast via [Submit Sealed Vote](council.md#user-content-submit-sealed-vote) must target valid candidate.
* The vote hasn't been revealed yet.
* `salt`'s length is not higher than `MAX_SALT_LENGTH`.

#### Effect

* Power proportional to the value staked for the vote is added to the candidate voted for.

### Recover Voting Stake

#### Parameters

No parameters.

#### Conditions

* A stake can be released only after the new council election ends.
* There exists a still locked voting stake associated with the authorizing account.

#### Effect

* A stake used for vote is unlocked.

### Recover Failed Candidacy Stake

#### Parameters

No parameters.

#### Conditions

* A stake can be released only after the new council election ends.
* There exists a still locked voting stake associated with the authorizing account.

#### Effect

* A stake used for vote is unlocked.

### Submit Candidacy Note

#### Parameters

| Name | Description |
| :--- | :--- |
| `membership_id` | Membership id uniquely identifying the user. |
| `note` | A note describing a candidate. |

#### Conditions

* The candidacy announcement or election period is running.
* The user is candidating in the current election.

#### Effect

* A note is associated with a candidate.

