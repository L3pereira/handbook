---
description: >-
  An immutable, auditable, public forum is the main communication and
  coordination forum among platform members.
---

# Forum

## Introduction

The forum is the primary place for community-wide asynchronous written communication about all topics relevant to the platform among members. It is hierarchically organized into a tree of categories, each with designated moderators responsible for policing and encouraging effective and beneficial interactions among members. The moderators are part of a designated forum working group, and the lead of that working group can decide what moderators are responsible for what categories. Categories contain subcategories, and threaded topic-based discussions, called _threads_, where any member can open a thread, and others can come and make replies in the form of _posts_. Some threads can also include a poll, allowing any member to weight in on a question.

## Roles

* **Member:** Members participate as normal forum users, creating and responding to threads, participating in polls, and so on.
* **Moderator:** Moderators are assigned to subsets of categories... \(stickied???\) hierarchical or just plain????
  * LIST ACTIONS THEY CAN DO.. they cant add or remove moderators!!!! Also cannot delete in root category, as the only moderator is root is lead....
* **Lead:** The forum lead is a member occupying the lead role in the forum working group. Beyond the normal working group lead obligations, this .. The lead can act as a moderator as well.
  * LIST ACTIONS THEY CAN DO
  * Is only actor to delete root categories, as lead is implict moderator of the anonymous root category.

## Concepts

### Archiving

Both categories and threads can be _archived,_ and updating this is the responsibility of the moderators. Archiving should be understood as a read-only mode for normal usage of the given part of the forum.



archived: directly or not,  ACTIVE: not archived in either sense

* direct or indirect, cant:
  * cant create category
* can still:
  * everythnge else: udpate archival status of decedant
  * udpate title & description & stickied threads

### Category

A category is defined by the following

* **Id:** A global unique post identifier, effectively the number of posts created in the forum prior to this one. From this one can infer of posts in a thread.
* **Parent:** An optional reference to a parent category. When not set, it indicates this is a root level category.
* **Title:** A human-readable category name.
* **Description:** A human-readable description.
* **Stickied Threads**: A list of threads in this category that have been designated as having long-run importance to the category.
* **Subcategories:** The categories with this category as its parent.
* **Threads:** The threads directly contained in this category, so not including subcategories in the count.
* **Moderators:** The moderator directly assigned to this category.
* **Archived:** Whether the category is archived or not. This impacts whether one can use the category, such as creating threads or posts directly, or recursively, within the category.

### Thread

A thread is defined by the following

* **Id:** A global unique post identifier, effectively the number of posts created in forum prior to this one. From this one can infer of posts in a thread.
* **Title:** The human readable title
* **Category:** The category within which the thread lives.
* **Author:** Member who created the thread.
* **Number of Posts:** The current number of posts in this thread.
* **Poll:** An optional poll for the thread, defined by the following
  * **Description:** A human readable description of what question is being polled.
  * **Deadline:** Some block before which is the only time anyone can participate in the poll.
  * **Alternatives:** A list of alternatives, each with its own explainer text, vote count and members who have voted in favor of it.
* **Archival Status:** Whether thread is archived or not.

### Post

A post in a thread is defined by the following

* **Id:** A global unique post identifier, effectively the number of posts created in the forum prior to this one. From this one can infer of posts in a thread.
* **Thread:** The thread 
* **Text:** xxxx
* **Author:** ....

## Constants

<table>
  <thead>
    <tr>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:center">Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>MAX_SUBCATEGORIES</code>
      </td>
      <td style="text-align:left">Maximum allowed subcategories in any
        <br />category to be created.</td>
      <td style="text-align:center"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MAX_THREADS_IN_CATEGORY</code>
      </td>
      <td style="text-align:left">Maximum number of threads in any category for
        <br />any new thread.</td>
      <td style="text-align:center"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MAX_POSTS_IN_THREAD</code>
      </td>
      <td style="text-align:left">
        <p>Maximum number of posts allowed in any thread for</p>
        <p>any new post.</p>
      </td>
      <td style="text-align:center"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MAX_MODERATORS_IN_CATEGORY</code>
      </td>
      <td style="text-align:left">
        <p>Maximum number of moderators allowed in a category</p>
        <p>for any new assignment of the moderator to category.</p>
      </td>
      <td style="text-align:center"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MAX_NUM_CATEGORIES</code>
      </td>
      <td style="text-align:left">
        <p>Maximum number of categories allowed in total for</p>
        <p>any new category to be created.</p>
      </td>
      <td style="text-align:center"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MAX_POLL_ALTERNATIVES</code>
      </td>
      <td style="text-align:left">Upper bound on the number of alternatives in a new poll.</td>
      <td style="text-align:center"><code>fill-in</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MAX_CATEGORY_TREE_DEPTH</code>
      </td>
      <td style="text-align:left">
        <p>Maximum category tree depth allowed for any</p>
        <p>new category to be created.</p>
      </td>
      <td style="text-align:center"><code>fill-in</code>
      </td>
    </tr>
  </tbody>
</table>

Notice that a lot of the limits are forward-looking. In the even to of a runtime upgrade, it may be that the limits are changed in a more restrictive direction, in which case it should not be expected that there is a migration that throws out the storage state, instead, the limit should only be understood to have bearing on future actions.

## Operations

### Create Category

**Parameters**

| Name | Description |
| :--- | :--- |
| `parent` | Optional parent category identifier. |
| `title` | Title of category. |
| `description` | Category description. |

#### Conditions

* Signer is working group lead.
* If provided, `parent` corresponds to valid category, and it is not directly or indirectly archived.
* Limit `MAX_NUM_CATEGORIES` is respected.
* Limit `MAX_CATEGORY_TREE_DEPTH` is respected.

#### Effect

A new category is created.

### Update Category Archival Status

**Parameters**

| Name | Description |
| :--- | :--- |
| `actor` | Either lead or working group identifier of moderator. |
| `category_id` | Category identifier. |
| `new_status` | Whether new status is archived or active |

#### Conditions

* Signer uses role account of `actor`.
* `category_id` corresponds to an existing category.
* If signer is moderator, then this moderator must be assigned to the category, or some ancestor category.
* The category has archival status different from `new_status`.

#### Effect

Archival status of category corresponding to `category_id` is updated to `new_status`.

### Update Category Title

**Parameters**

| Name | Description |
| :--- | :--- |
| `actor` | Either lead or working group identifier of moderator. |
| `category_id` | Category identifier. |
| `new_title` | New title for category. |

#### Conditions

* Signer uses role account of `actor`.
* `category_id` corresponds to an existing category.
* If signer is moderator, then this moderator must be assigned to the category, or some ancestor category.

#### Effect

The title of the category corresponding to `category_id` is set to `new_title`.

### Update Category Description

**Parameters**

| Name | Description |
| :--- | :--- |
| `actor` | Either lead or working group identifier of moderator. |
| `category_id` | Category identifier. |
| `new_description` | New description for category. |

#### Conditions

* Signer uses role account of `actor`.
* `category_id` corresponds to an existing category.
* If signer is moderator, then this moderator must be assigned to the category, or some ancestor category.

#### Effect

The description of the category corresponding to `category_id` is set to `new_description`.

### Set Category Stickied Threads

**Parameters**

| Name | Description |
| :--- | :--- |
| `actor` | Either lead or working group identifier of moderator. |
| `category_id` | Category identifier. |
| `threads` | List of thread identifiers. |

#### Conditions

* Signer uses role account of `actor`.
* `category_id` corresponds to an existing category.
* If signer is moderator, then this moderator must be assigned to the category, or some ancestor category.
* All identifiers `threads`corresponding to existing threads.

#### Effect

The stickied threads of the category corresponding to `category_id` is set to `threads`.

### Update Category Membership of Moderator

**Parameters**

| Name | Description |
| :--- | :--- |
| `category_id` | Category identifier. |
| `moderator` | Working group identifier of moderator. |
| `is_member` | Whether moderator should be member. |

#### Conditions

* Signer is working group lead.
* `category_id` corresponds to an existing category.

_Note: There is no check that the provided moderator identifier corresponds to a genuine working group moderator._

#### Effect

If `is_member` is true, then the `moderator` identifier is added to the category moderators for category corresponding to `category_id` so long as the `MAX_MODERATORS_IN_CATEGORY` limit is respected. If `is_member`is not true, then the `moderator` is removed if present.

### Delete Category

**Parameters**

| Name | Description |
| :--- | :--- |
| `actor` | Either lead or working group identifier of moderator. |
| `category_id` | Category identifier. |

#### Conditions

* Signer uses role account of `actor`.
* `category_id` corresponds to an existing category.
* There are no threads in the category.
* There are no subcategories in the category.
* If the parent of the category is the root, then `actor` must be the lead, otherwise for other categories `actor` must be lead, or moderator assigned to the category, or some ancestor category.

#### Effect

The category is dropped.

### Create Thread

**Parameters**

| Name | Description |
| :--- | :--- |
| `member_id` | Identifier of member. |
| `category_id` | Category identifier. |
| `title` | Thread title. |
| `text` | Body text of thread. |
| `poll` | Optional poll for thread. |

#### Conditions

* Signer uses role account of member corresponding to `member_id`.
* `category_id` corresponds to an existing category.
* Limit `MAX_THREADS_IN_CATEGORY` is respected.
* xx mutable category
* when poll set: alterantive valid, poll is valid?

#### Effect

A...

### Update Thread Archival Status

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### Edit Thread Title

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### Move Thread

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### Moderate Thread

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### Delete Thread

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### Create Post

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### Edit Post

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### React to Post

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### Moderate Post

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### Delete Post

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

### Vote On Poll

**Parameters**

| Name | Description |
| :--- | :--- |
| \`\` |  |

#### Conditions

* Signer matches controller account 

#### Effect

A...

## Examples

xxx





