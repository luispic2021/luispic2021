---
layout: post
title: "Ticket Hierarchy in JIRA"
date: 2025-02-08
categories: product-management agile
---

## Defining Ticket Hierarchy in JIRA

In any software project, maintaining a **clear ticket hierarchy** is essential for organizing work, ensuring accountability, and driving efficient development. In this post, I’ll break down the different ticket types used in **JIRA** and how I structure them for my **Permabot project**.

## **Understanding JIRA Ticket Types**

JIRA provides several ticket types, each serving a specific purpose. Here’s how I categorize them:

- **Epic** – A high-level deliverable representing a scoped user value.
- **Story (User Story)** – A feature or user interaction that enhances the product.
- **Bug** – An unintended behavior that needs fixing.
- **Task** – Non-development work, such as data backfilling or customer interactions.
- **Subtask** – The smallest unit of work, breaking down a Story, Bug, or Task.

A well-structured hierarchy ensures **clarity in ownership, progress tracking, and smooth execution**.

---

## **Example: Permabot’s Ticket Hierarchy**

Here’s how I structure JIRA tickets for **Permabot**:

1. **Epic:** Clearly defines end-user value and deliverables.
2. **Stories:** Break down the epic into actionable work.
3. **Bugs:** Track issues that arise during testing.

### **JIRA Ticket Breakdown Example**

For Permabot v2.3.1, the Epic started with these high-level goals:

- Improve logging to capture delta values for option legs, enabling better risk assessment and strategy adjustments.
- Implement dynamic contract sizing to limit potential losses to a maximum of 3% per trade on a $21,000 account.
- Enhance logging for HTTP errors to improve debugging and system reliability.

Here’s how my JIRA hierarchy appears in the timeline view:

![JIRA Ticket Hierarchy](/assets/images/jira-timeline-permabot-v2.3.1-done.jpg)

During development, the number of features expanded due to **scope creep**. While this is natural, as deeper exploration surfaces additional issues and opportunities, it’s crucial for a **product manager to monitor and manage scope effectively**. As a general rule, I allow developers to add necessary stories when changes to existing code are required to deliver on the original goals. For instance, before enhancing HTTP error logging, it may be necessary to implement basic logging first. However, external stakeholders should not bypass the product manager when requesting additional features directly from developers.

Another key takeaway is that the Epic initially had **zero bugs**, but as development progressed, unexpected behaviors surfaced—especially given that **the software is still evolving** despite being at version v2.3+. This is normal. Logging these issues ensures they are addressed appropriately, whether as immediate fixes (**P0 priority**) or for future consideration (**P1+ priority**). I’ll cover prioritization strategies in a separate post.

---

## **Example Changelog: Permabot v2.3.1**

Here’s a real changelog from one of Permabot’s releases:

```
## [2.3.1] - 2024-XX
### Added
- Configured max delta to 0.20 and stop loss is 1.75x
- Created Utils function to log response / requests for debugging
- Dynamic sizing defaulting to 3% max loss on 21000 act
- Added push notifications for active monitoring, alerting in event of 401
- WBPR Bot: Refactored William's Brown Price Reversion strategy supporting >1 daily entries
- Introduction of Perma Journal
- WBPR Bot: standardized notifications
- Json dump utils with logger support

### Fixed
- monitor_and_exit loop continues forever when there are http errors, in case connection comes back.
- WBPR Bot: no trade days end the bot
```

---

## **Final Thoughts**
A well-defined **ticket hierarchy** in JIRA helps ensure a structured and scalable software development process. By clearly defining **Epics, Stories, Bugs, and Tasks**, teams can streamline execution and maintain better control over project progress.

**How do you structure your JIRA tickets? Let’s connect and share insights!**
