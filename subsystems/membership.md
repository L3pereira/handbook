---
description: >-
  Memberships are the stable identifier under which actors occupy roles, submit
  proposals and communicate on the platform.
---

# Membership

## Introduction

A membership is a representation of an actor on the platform, and it exist to serve the following purposes

* **Profile:** A membership has an associated rich profile that includes information that support presenting the actor in a human friendly way in applications, much more so than raw accounts in isolation.
* **Reputation:** Facilitates the consolidation of all activity under one stable identifier, allowing an actor to invest in the reputation of a membership through prolonged participation with good conduct. This gives honest and competent actors a practical way to signal quality, and this quality signal is a key screening parameter allowing entry into more important and sensitive activities. While nothing technically prevents an actor from registering for multiple memberships, the value of doing a range of activities under one membership should be greater than having it fragmented, since reputation, in essence, increases with the length and scope of the history of consistent good conduct.

It's important to be aware that a membership is not an account, but a higher level concept that involves accounts for authentication. The membership subsystem is responsible for storing and managing all memberships on the platform, as well as enabling the creation of new memberships, and the terms under which this may happen.

## Concepts

### Membership

A membership includes the following

* **Id:** A unique immutable non-negative integer identifying the member, automatically assigned when membership is created.
* **Root Account:** A required account that is used only to update the controller account. Need not be unique across members, but in practice probably will be.
* **Controller Account:** A required account that is used to authenticate as the member, both in this and other parts of the platform. Need not be unique across members, but in practice probably will be.
* **Name:** A human readable mutable string.
* **Handle:** A unique mutable string handle.
* **Invites:** A mutable non-negative integer that represents how many invitations this member has.
* **Verified:** A mutable boolean indicator that reflects whether the implied real world identity in the profile corresponds to the true actor behind the membership.
* **Avatar:** A mutable URI for an avatar image.
* **About:** A mutable human readable text description.
* **Founding Member**: A signifier that this member holds some specific historical significance to the launch of the platform. This value will be stored in the chain state when mainnet launches, but for now, since we want to grant founding member status on an ongoing member through a SUDO call, this is in history.
* **Staking Accounts:** A set of accounts that have been bound to this membership for the purpose of holding staked funds. One account can only be used to stake for at most two separate purposes simultaneously, and one of them has to be an election related purpose, i.e. voting or council candidacy. One account can only be a staking account for a single member, and once associated in this way, it cannot be de-associated and associated with another member.

### Working Group

The membership subsystem has a working group. The purpose of the group is to effectively distribute invitation quotas and verified status. The lead, called the _membership lead_  has the extra task of refreshing the quotas to workers, which they can in turn then distribute to other members. Workers are referred to as _membership evangelists_.

### Invitations

The long-term objective is to have most memberships established by being purchased, however, currently the various costs associated with gaining access to a digital asset are considerable. As a result, person-to-person invitations is an alternative mechanism for giving new community members direct access to participate on the platform. This works by giving each person an _invite quota_, that is a certain number of outstanding invitations available at any given time. The quota is initially set when buying a membership, but it can also be increased by having another member transfer some of theirs. The membership lead has at regular intervals of length `INVITATION_BUDGET_PERIOD`, called _invitation budget periods_, have `invitation_budget_increments` added to their quota.  
  
When a member is invited, they are also credited some minimal quantity of funds to their controller account so that they can engage in some initial set of activities. These funds are however only spendable on transaction fees, nothing else, such as transferring to another member.

## Constants

The following constants are hard coded into the system, they can only be updated with a runtime upgrade.

### General

| Name | Description | Value |
| :--- | :--- | :--- |
| `INVITATION_BUDGET_PERIOD` | The number of blocks an invitation  budget period lasts. | `fill-in` |
| `INVITE_LOCK_ID` | The identifier value for the lock applied to root account of a new member | `fill-in` |

### Working Group

| Name | Value |
| :--- | :--- |
| `MAX_NUMBER_OF_WORKERS` | `fill-in` |
|  | . |

## Parameters

Parameters are on-chain values that can be updated through the proposal system in order to alter the constraints and functionality of the membership system.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>membership_price</code>
      </td>
      <td style="text-align:left">The price of buying a membership.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>referral_cut</code>
      </td>
      <td style="text-align:left">The referral cut of the membership price diverted to a referrer when buying
        a membership. Must be no greater than <code>membership_price</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>default_invite_count</code>
      </td>
      <td style="text-align:left">The default number of invitations set for a new bought membership.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>invited_initial_balance</code>
      </td>
      <td style="text-align:left">The initial balance credited to an invited members controller account.</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>invitation_budget_increments</code>
        </p>
        <p></p>
      </td>
      <td style="text-align:left">The number of invites added to the quota of the lead at each invitation
        budget period.</td>
    </tr>
  </tbody>
</table>

## Operations

### Buy a Membership

**Parameters**

| Name | Description |
| :--- | :--- |
| `root_account` | To be root account of membership. |
| `controller_account` | To be controller account of membership. |
| `name` | To be name of membership. |
| `handle` | To be handle of membership. |
| `avatar_uri` | To be avatar URI of membership. |
| `about` | To be about field of membership. |
| `referer_id` | Optional identifier of some existing member. |

#### Conditions

* Free balance of the signer account exceeds `membership_price`. 
* `handle` must be unique among all existing handles.
* If provided, `referer_id` must correspond to an existing member.

#### Effect

* A new membership is created with the provided information, and initial invites set to `default_invite_count`.
* If `referer_id` is provided, then the corresponding member has`referral_cut` credited to their controller account and `membership_price - referral_cut` is burned, otherwise `membership_price` is burned.

### Invite a Member

**Parameters**

| Name | Description |
| :--- | :--- |
| `member_id` | Identifier of inviting member. |
| `root_account` | To be root account of membership. |
| `controller_account` | To be controller account of membership. |
| `name` | To be name of membership. |
| `handle` | To be handle of membership. |
| `avatar_uri` | To be avatar URI of membership. |
| `about` | To be about field of membership. |

#### Conditions

* Signer matches controller account of member corresponding to `member_id`.
* Invitation quota of member is non-zero.
* `handle` must be unique among all existing handles.

#### Effect

* A new membership is created with the provided information, and initial invites set to zero, and the root account is credited with`invited_initial_balance`.
* `controller_account` is encumbered with lock with ID `INVITE_LOCK_ID` and of size `invited_initial_balance`  that only allows transaction fees, and credited with the same amount of tokens.
* Invite quota of member corresponding to `member_id` is decremented.

### Update Profile

**Parameters**

| Name | Description |
| :--- | :--- |
| `member_id` | Identifier of member wishing to update profile. |
| `name` | Optional new name for membership. |
| `handle` | Optional new handle for membership. |
| `avatar_uri` | Optional new avatar URI for membership. |
| `about` | Optional new about field for membership. |

#### Conditions

* Signer matches controller account of member corresponding to `member_id`.  
* if set `handle` must be unique among all existing handles.
* at least one of the optional fields must be set.

#### Effect

Profile of member corresponding to `member_id` is updated with new field values.

### Transfer Invites

**Parameters**

| Name | Description |
| :--- | :--- |
| `member_id` | Identifier of member wishing to send invites. |
| `recipient_member_id` | Identifier of member to be credited with invites. |
| `number_of_invites` | Number of invites to transfer. |

#### Conditions

* Signer matches controller account member corresponding to  `member_id`.
* `recipient_member_id` corresponds to a existing recipient member.
* `number_of_invites` is no greater than invitation quota of sender.

#### Effect

`number_of_invites` is transferred from invite quota of sender to recipient.

### Update Accounts

**Parameters**

| Name | Description |
| :--- | :--- |
| `member_id` | Identifier for member wishing to update accounts. |
| `root_account` | Optional new root account for membership. |
| `controller_account` | Optional new controller account for membership. |

#### Conditions

* Signer matches controller account member corresponding to  `member_id`.
* At least one new account is provided.

#### Effect

Update provided accounts on member.

### Update Verified Status

**Parameters**

| Name | Description |
| :--- | :--- |
| `worker_id` | Identifier of membership evangelist. |
| `member_id` | Identifier of member to have status updated. |
| `is_verified` | New status of member. |

#### Conditions

* Signer matches controller account of worker corresponding to `worker_id`.
* `member_id` corresponds to some existing member.

#### Effect

Verification status of member is set to `is_verified`.

### Bind Staking Account

**Parameters**

| Name | Description |
| :--- | :--- |
| `member_id` | Identifier of member to bind account. |
| `account` | Account to be bound. |

#### Conditions

* Signer matches controller account of member corresponding to `member_id`.
* `account` is not already bound to any other member.

#### Effect

`account` is bound to member.

