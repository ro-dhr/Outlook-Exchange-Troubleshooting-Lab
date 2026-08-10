# Microsoft 365 / Outlook & Exchange Help Desk Lab

Ten Tier 1 support tickets, diagnosed and resolved end-to-end in a live Microsoft 365 tenant. Built around a simulated MSP service desk environment; tenant, users, licensing, shared mailboxes, and distribution lists all provisioned from scratch. Each one mirrors a real service desk queue item (mailbox permissions, distribution list failures, missing mail), worked through the same reproduce → diagnose → fix → verify process a help desk tech would follow.

## Environment

| | |
|---|---|
| **Name** | `RoodeeMSP` |
| **Tenant** | `roodeeMSP.onmicrosoft.com` (Office 365 E5 trial) |
| **Admin tools used** | Microsoft 365 admin center, Exchange admin center (EAC), Outlook on the web |
| **Test users** | John Stanley, Kenneth Green, Maria Garcia, Rooble Dahir (admin), Sandra Clark |
| **Shared mailbox** | Service Desk (`servicedesk@roodeeMSP.onmicrosoft.com`) |
| **Distribution list** | Client Services (`clientservices@roodeeMSP.onmicrosoft.com`) |
| **External test account** | A personal Gmail address, used to simulate a client emailing in from outside the organization |

## Tickets (In progress, 3/10 Completed)

| # | Ticket | Area |
|---|---|---|
| 01 | [Shared mailbox access request](#ticket-01--shared-mailbox-access-request) | Permissions |
| 02 | [Distribution list membership](#ticket-02--distribution-list-not-receiving-mail) | Mail flow |
| 03 | [Missing emails](#ticket-03--missing-emails) | Mail flow |

---

## Ticket 01 — Shared Mailbox Access Request

**Scenario:** A staff member (John Stanley) needs access to the Service Desk shared mailbox to help manage incoming client requests, but currently can't open it.

### Reproduce
Signed in as John Stanley and attempted to open the Service Desk mailbox via *Open another mailbox*. The mailbox failed to load with an `AccessDeniedException`.

![Baseline — no delegation assigned](screenshots/ticket01-00-baseline-no-delegation.png)
![Access denied error](screenshots/ticket01-01-jstanley-accessdenied.png)

### Diagnose
Checked the Service Desk mailbox's **Delegation** tab in the Exchange admin center — both **Send as** and **Read and manage (Full Access)** were at 0, confirming John had no permissions assigned.

### Fix — Part 1: Full Access
Granted **Full Access** to John Stanley under the mailbox's Delegation settings.

![Full Access granted](screenshots/ticket01-02-fullaccess-granted.png)
![Confirmation](screenshots/ticket01-03-fullaccess-confirmation.png)

After allowing time for the change to propagate, John was able to open the mailbox and view its folders (Inbox, Sent Items) — confirming Full Access was working.

![Reopening the mailbox](screenshots/ticket01-04-reopening-mailbox.png)
![Mailbox opens successfully](screenshots/ticket01-05-mailbox-opens-success.png)
![Sent Items visible — proof of read access](screenshots/ticket01-06-sentitems-proof.png)

### Fix — Part 2: Send As
Full Access alone allows reading and managing a mailbox, but not sending *as* it. When John tried to compose a new message from the Service Desk address, it failed with a permissions error.

![Send As denied](screenshots/ticket01-07-sendas-denied.png)

Granted **Send as** permission separately in the same Delegation tab.

![Granting Send As](screenshots/ticket01-08-sendas-granting.png)
![Send As confirmation](screenshots/ticket01-09-sendas-confirmation.png)

### Verify
John sent a test email from the Service Desk mailbox to Kenneth Green. The message was delivered successfully, showing Service Desk as the sender.

![Sending as Service Desk](screenshots/ticket01-10-sending-as-servicedesk.png)
![Kenneth Green receives the email](screenshots/ticket01-11-kgreen-receives-verification.png)

**Root cause:** No delegation permissions had been assigned to the mailbox. **Resolution:** Full Access and Send As are separate permissions and both needed to be granted individually to fully resolve the request.

---

## Ticket 02 — Distribution List Not Receiving Mail

**Scenario:** A client reports that emails sent to the Client Services team aren't reaching anyone.

### Reproduce
The Client Services distribution list had zero members. An external test (from Gmail) and an internal test (from John Stanley) were both sent to `clientservices@roodeeMSP.onmicrosoft.com` — neither reached Sandra Clark, who was expected to be receiving list mail.

![Baseline — empty distribution list](screenshots/ticket03-00-baseline-empty-dl.png)
![External sender test — fails](screenshots/ticket03-01-external-sends-fails.png)
![Internal sender test — fails](screenshots/ticket03-02-internal-sends-fails.png)
![Sandra's inbox — nothing received](screenshots/ticket03-03-sandra-no-receipt.png)

### Diagnose
Checked the distribution list's **Membership** tab in the Exchange admin center — confirmed it had 0 members, so there was nowhere for incoming mail to be delivered.

### Fix — Part 1: Membership
Added Sandra Clark as a member of the Client Services list.

![Adding Sandra Clark](screenshots/ticket03-04-adding-sandra.png)
![Confirmation](screenshots/ticket03-05-sandra-added-confirmation.png)

Retested with an internal sender — mail now reached Sandra successfully, confirming membership was the first issue.

![Internal delivery now works](screenshots/ticket03-06-internal-now-works.png)

### Fix — Part 2: External Delivery
The external (Gmail) test still failed after membership was fixed. Checking the list's **Delivery management** settings under Sender options showed it was restricted to internal senders only — the default setting for a new distribution list.

![Delivery management restricted to internal only](screenshots/ticket03-07-delivery-management-internal-only.png)

Enabled **"Allow messages from people inside and outside my organization."**

![External senders enabled](screenshots/ticket03-08-external-senders-enabled.png)

### Verify
Resent the test message from Gmail. It was delivered to Sandra's inbox successfully.

![External retest sent](screenshots/ticket03-09-external-retest-sent.png)
![External delivery verified](screenshots/ticket03-10-external-verification.png)

**Root cause:** Two separate issues — the list had no members, and it was also configured to reject external senders by default. **Resolution:** Both had to be identified and fixed independently; fixing only one would have left the client's original complaint unresolved.

---

## Ticket 03 — Missing Emails

**Scenario:** Maria Garcia reports that she isn't receiving emails from a specific external contact, despite them confirming the messages were sent.

### Reproduce
A test email was sent from the external Gmail account to Maria Garcia. It did not appear in her inbox.

![Test email sent](screenshots/ticket04-01-test-email-sent.png)
![Missing from inbox](screenshots/ticket04-02-inbox-missing-email.png)

### Diagnose
Rather than assuming a spam or delivery issue, ran a **message trace** in the Exchange admin center for the sender/recipient pair.

![Message trace configuration](screenshots/ticket04-03-messagetrace-config.png)

The trace showed the message was successfully **delivered** to Maria's mailbox — ruling out mail flow or filtering as the cause and pointing to something happening inside the mailbox itself after delivery.

![Message trace — delivered](screenshots/ticket04-04-messagetrace-delivered.png)

Checked Maria's Deleted Items folder and found the message there.

![Message found in Deleted Items](screenshots/ticket04-05-found-in-deleted-items.png)

Reviewing her inbox rules revealed a rule silently moving messages from that sender straight to Deleted Items.

![Culprit rule identified](screenshots/ticket04-06-culprit-rule-found.png)

### Fix
Disabled the rule.

![Rule disabled](screenshots/ticket04-07-rule-disabled.png)

### Verify
Sent a new test email from the same external address. It landed directly in Maria's inbox as expected.

![Verified in inbox](screenshots/ticket04-08-verification-inbox.png)

**Root cause:** A mailbox rule was silently redirecting mail from the sender to Deleted Items. **Resolution:** Message trace confirmed delivery was succeeding at the server level, which narrowed the investigation to client-side mailbox rules rather than mail flow or spam filtering.

---

## Notes on Tooling

All fixes in this lab were made through the Exchange admin center GUI. In a production MSP environment, the same changes are commonly scripted via PowerShell for speed and consistency across multiple tenants — for example:

```powershell
# Ticket 01 equivalent
Add-MailboxPermission -Identity "Service Desk" -User jstanley -AccessRights FullAccess
Add-RecipientPermission -Identity "Service Desk" -Trustee jstanley -AccessRights SendAs

# Ticket 03 equivalent
Add-DistributionGroupMember -Identity "Client Services" -Member sclark
Set-DistributionGroup -Identity "Client Services" -RequireSenderAuthenticationEnabled $false
```

## Planned Additions

Further tickets from the original repo are planned for this lab, covering onboarding/provisioning, quarantine release, mail forwarding, and automatic replies.
