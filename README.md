# Microsoft 365 / Outlook & Exchange Help Desk Lab

Eight Tier 1 support tickets, diagnosed and resolved end-to-end in a live Microsoft 365 tenant. Built around a simulated MSP service desk environment; tenant, users, licensing, shared mailboxes, and distribution lists all provisioned from scratch. Each one mirrors a real service desk queue item (mailbox permissions, distribution list failures, missing mail), worked through the same reproduce → diagnose → fix → verify process a help desk tech would follow.

## Environment

| | |
|---|---|
| **Name** | `RoodeeMSP` |
| **Tenant** | `roodeeMSP.onmicrosoft.com` (Office 365 E5 trial) |
| **Admin tools used** | Microsoft 365 admin center, Exchange admin center (EAC), Entra admin center, Outlook on the web |
| **Test users** | John Stanley, Kenneth Green, Maria Garcia, Rooble Dahir (admin), Sandra Clark, Jordan Miller |
| **Shared mailboxes** | Service Desk (`servicedesk@roodeeMSP.onmicrosoft.com`), Sales (`sales@roodeeMSP.onmicrosoft.com`) |
| **Distribution list** | Client Services (`clientservices@roodeeMSP.onmicrosoft.com`) |
| **External test account** | A personal Gmail address, used to simulate a client emailing in from outside the organization |

## Tickets 

| # | Ticket | Area |
|---|---|---|
| 01 | [Shared mailbox access request](#ticket-01--shared-mailbox-access-request) | Permissions |
| 02 | [Distribution list membership](#ticket-02--distribution-list-not-receiving-mail) | Mail flow |
| 03 | [Missing emails](#ticket-03--missing-emails) | Mail flow |
| 04 | [New employee onboarding](#ticket-04--new-employee-onboarding) | Provisioning |
| 05 | [Calendar sharing permissions](#ticket-05--calendar-sharing-permissions) | Permissions |
| 06 | [Quarantined message release](#ticket-06--quarantined-message-release) | Security |
| 07 | [Temporary mail forwarding](#ticket-07--temporary-mail-forwarding) | Mail flow |
| 08 | [Automatic replies not reaching external senders](#ticket-08--automatic-replies-not-reaching-external-senders) | Client |

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

## Ticket 04 — New Employee Onboarding

**Scenario:** A new hire, Jordan Miller, is joining the Sales team and needs to be fully provisioned — licensed, given a mailbox, added to the appropriate shared resources, and secured with MFA — before their first day.

### Provision the account
Created Jordan Miller as a new user and assigned an Office 365 E5 license.

![User created](screenshots/ticket06-01-user-created.png)
![License assigned](screenshots/ticket06-02-license-assigned.png)

### Set up department mailbox access
Since Jordan's role is Sales-facing, a new **Sales** shared mailbox was created and delegated to him, following the same permission model used in Ticket 01 (Full Access + Send As).

![Full Access granted on Sales](screenshots/ticket06-03-sales-fullaccess-granted.png)
![Send As granted on Sales](screenshots/ticket06-04-sales-sendas-granted.png)
![Sales mailbox opened as Jordan](screenshots/ticket06-05-sales-mailbox-opened.png)

Verified by sending a test email as Sales to Kenneth Green.

![Sending as Sales](screenshots/ticket06-06-sending-as-sales.png)
![Kenneth receives the email from Sales](screenshots/ticket06-07-kenneth-receives-verification.png)

### Add to Client Services distribution list
Added Jordan to the Client Services list so he receives client-facing mail alongside Sandra.

![Added to Client Services](screenshots/ticket06-08-added-to-clientservices.png)

Verified with a test email from the external Gmail account.

![External client sends to Client Services](screenshots/ticket06-09-external-sends-to-clientservices.png)
![Jordan receives the email](screenshots/ticket06-10-jordan-receives-clientservices-mail.png)

### Enable MFA
Enabled per-user MFA for Jordan's account so it's enforced on his first sign-in.

![Enabling MFA](screenshots/ticket06-11-enabling-mfa.png)
![MFA enabled confirmation](screenshots/ticket06-12-mfa-enabled-confirmation.png)

On his next sign-in, Jordan was prompted to register for MFA and set up Microsoft Authenticator.

![MFA registration prompt](screenshots/ticket06-13-mfa-registration-prompt.png)
![Authenticator QR code](screenshots/ticket06-14-mfa-qr-code.png)
![MFA setup complete](screenshots/ticket06-15-mfa-setup-complete.png)
![Signed in successfully after MFA setup](screenshots/ticket06-16-signed-in-post-mfa.png)

### Extra: Mailbox Storage Adjustment
As an additional provisioning step, Jordan's mailbox storage quota was reduced from the default to 1024 GB, then to 200 GB.

![Changing storage limit](screenshots/ticket06-17-storage-limit-change.png)
![Storage confirmed at 200GB](screenshots/ticket06-18-storage-confirmed-200gb.png)

**Resolution:** Full onboarding completed end to end — license assigned, department shared mailbox provisioned and delegated, distribution list membership added, MFA enforced, and mailbox storage adjusted for role requirements. Each piece was verified independently rather than assumed to work once configured.

---

## Ticket 05 — Calendar Sharing Permissions

**Scenario:** Sandra Clark needs full visibility into Kenneth Green's calendar to help coordinate client meetings, but she can currently only see whether he's free or busy — not the meeting details.

### Reproduce
Signed in as Sandra Clark and opened Kenneth Green's calendar. Only free/busy blocks were visible; no subject, location, or details for any event.

![Sandra can only see busy/free blocks](screenshots/ticket05-00-baseline-busy-free-only.png)

### Diagnose
Checked Kenneth's mailbox delegation settings in the Exchange admin center first, since a permissions issue was assumed to be delegation-related. Send As, Send on Behalf, and Full Access were all correctly configured — ruling out mailbox delegation as the cause.

![Ruling out mailbox delegation in EAC](screenshots/ticket05-01-eac-delegation-ruled-out.png)

This narrowed it down: calendar visibility is controlled separately from mailbox delegation, through the calendar's own sharing permissions in Outlook.

### Fix — Part 1: Calendar Sharing
From Kenneth's calendar, opened **Sharing and permissions** and updated Sandra Clark's access level from the default to **Can view all details**.

![Granting Sandra "Can view all details"](screenshots/ticket05-02-calendar-sharing-can-view-all-details.png)

Sandra could then see full event details on Kenneth's calendar, confirming the sharing-level change resolved the visibility issue.

![Sandra now sees full event details](screenshots/ticket05-03-sandra-views-full-details.png)

### Fix — Part 2: Delegate Access
The original request also included Sandra being able to create and manage events on Kenneth's calendar on his behalf, not just view it — which requires **delegate** access rather than just sharing permissions. Added Sandra as a delegate from the same Sharing and permissions dialog.

![Granting delegate access to Sandra](screenshots/ticket05-04-delegate-access-granted.png)

### Verify
Sandra created a test event directly on Kenneth's calendar to confirm delegate access was working.

![Sandra creates an event on Kenneth's calendar](screenshots/ticket05-05-sandra-creates-event.png)

Signed in as Kenneth and confirmed the event appeared on his own calendar.

![Kenneth sees the event Sandra created](screenshots/ticket05-06-kenneth-sees-sandras-event.png)

**Root cause:** Calendar visibility and mailbox delegation are two separate permission systems in Exchange/Outlook — delegation being correctly configured didn't mean calendar sharing was. **Resolution:** Calendar sharing level had to be raised to "Can view all details" for visibility, and delegate access granted separately for Sandra to manage events on Kenneth's behalf.

---

## Ticket 06 — Quarantined Message Release

**Scenario:** Maria Garcia says she's expecting an email from an external contact, but it never arrived — not in her inbox, and not in Junk.

### Reproduce
Confirmed the message wasn't in Maria's inbox or Junk Email folder.

![Missing from inbox](screenshots/ticket06-00-missing-from-inbox.png)
![Also missing from Junk](screenshots/ticket06-01-missing-from-junk.png)

### Diagnose
Rather than guessing, ran a **message trace** in the Exchange admin center for the sender/recipient pair to see what actually happened to the message server-side.

![Message trace configuration](screenshots/ticket06-02-messagetrace-config.png)

The trace showed the message was flagged as **spam** and sent to quarantine — not delivered, and not rejected either.

![Trace result: identified as spam, sent to quarantine](screenshots/ticket06-03-messagetrace-result-quarantined.png)

Located the message directly in Microsoft Defender's **Quarantine** view to confirm.

![Message found in quarantine](screenshots/ticket06-04-found-in-quarantine.png)
![Additional quarantine details](screenshots/ticket06-05-quarantine-details.png)

### Fix
Released the message from quarantine to Maria's mailbox.

![Releasing the message from quarantine](screenshots/ticket06-06-released-from-quarantine.png)

### Verify
Maria could now see the email in her inbox.

![Maria sees the email](screenshots/ticket06-07-maria-sees-email.png)

Re-ran the message trace to confirm the delivery status updated to delivered.

![Message trace now shows delivered](screenshots/ticket06-08-messagetrace-delivered-confirmed.png)

**Root cause:** The anti-spam policy flagged the external message and routed it to quarantine before it ever reached Maria's mailbox, so it wouldn't appear in Junk either — quarantine sits ahead of mailbox delivery entirely. **Resolution:** Message trace was the key diagnostic step here — it confirmed the message had left the sender successfully and pinpointed exactly where it stalled, instead of guessing between spam filtering, rules, or a delivery failure.

---

## Ticket 07 — Temporary Mail Forwarding

**Scenario:** John Stanley is going to be out of the office for two weeks and wants Sandra Clark to receive a copy of everything sent to him during that period, without permanently changing his mailbox setup.

### Set Up the Rule
**Why a transport rule instead of simple forwarding:** A user-level forwarding rule runs indefinitely until someone remembers to disable it, and auto-forwarding is often restricted by admins since it's a common way attackers exfiltrate mail from compromised accounts. A transport rule solves both: it's time-boxed to expire on its own, and BCCing a copy (rather than forwarding) means John still gets everything in his own inbox too.

Created a new **transport (mail flow) rule** in the Exchange admin center: if the recipient is John Stanley, BCC the message to Sandra Clark.

![Setting up the transport rule conditions](screenshots/ticket07-00-transport-rule-conditions.png)

Since this needed to be temporary, set the rule to activate and automatically deactivate on specific dates rather than running indefinitely.

![Setting the rule's active date range](screenshots/ticket07-01-transport-rule-dates.png)

Rule was created successfully.

![Rule created](screenshots/ticket07-02-rule-created.png)

### Reproduce (the issue)
Sent a test email to John to confirm the rule was working — but Sandra never received a copy. Checked the rule's status and found it had been created in a **disabled** state by default.

![Rule exists but is disabled](screenshots/ticket07-03-rule-disabled-issue.png)

### Fix
Enabled the rule.

![Rule enabled](screenshots/ticket07-04-rule-enabled.png)

### Verify
Resent a test email to John. Both John and Sandra received it, confirming the BCC copy was working as expected.

![John receives the email](screenshots/ticket07-05-john-received.png)
![Sandra also receives a copy](screenshots/ticket07-06-sandra-received-bcc.png)

**Root cause:** New transport rules aren't enabled by default when created — the rule was configured correctly from the start but never actually active. **Resolution:** Verifying a rule's enabled/disabled status is a simple but easy-to-miss check after configuration, and this ticket was a reminder to confirm the whole pipeline (created → enabled → date range) rather than assuming "created" means "active."

---

## Ticket 08 — Automatic Replies Not Reaching External Senders

**Scenario:** Maria Garcia set up an out-of-office automatic reply. Colleagues confirm they're getting it, but a client emailing from outside the organization says they never received one.

### Baseline
Confirmed automatic replies were working correctly for internal senders.

![Automatic reply working for internal senders](screenshots/ticket08-00-internal-autoreply-works.png)

### Reproduce
Sent a test email to Maria from an external Gmail account.

![External email sent to Maria before the fix](screenshots/ticket08-01-external-email-sent-before-fix.png)

Confirmed the message itself was delivered successfully — this wasn't a mail flow issue.

![Proof the external email was received](screenshots/ticket08-02-maria-received-external-email.png)

However, no automatic reply came back to the external sender.

![External sender receives no automatic reply](screenshots/ticket08-03-external-sender-no-autoreply.png)

### Diagnose
Checked Maria's Automatic Replies settings and found **"Send replies outside your organization"** was unchecked — automatic replies were only configured to send internally.

![External automatic replies disabled](screenshots/ticket08-04-external-autoreply-disabled.png)

### Fix
Enabled external automatic replies with a separate message for outside senders.

![Enabling external automatic replies](screenshots/ticket08-05-enabling-external-autoreply.png)

### Verify
Resent a test email from the external Gmail account.

![External email resent after the fix](screenshots/ticket08-06-external-email-resent.png)

The external sender received the automatic reply this time.

![External sender receives the automatic reply](screenshots/ticket08-07-external-autoreply-received.png)

**Root cause:** Automatic replies have two independent toggles — internal and external — and only the internal one was enabled. **Resolution:** Confirming the underlying message was actually delivered first (rather than assuming a mail flow problem) narrowed this down quickly to an Automatic Replies configuration issue rather than anything server-side.

---

## Notes on Tooling

All fixes in this lab were made through the Exchange admin center, Microsoft 365 admin center, and Entra admin center GUIs. In a production MSP environment, the same changes are commonly scripted via PowerShell for speed and consistency across multiple tenants — for example:

```powershell
# Ticket 01 equivalent
Add-MailboxPermission -Identity "Service Desk" -User jstanley -AccessRights FullAccess
Add-RecipientPermission -Identity "Service Desk" -Trustee jstanley -AccessRights SendAs

# Ticket 02 equivalent
Add-DistributionGroupMember -Identity "Client Services" -Member sclark
Set-DistributionGroup -Identity "Client Services" -RequireSenderAuthenticationEnabled $false

# Ticket 04 equivalent
New-Mailbox -Shared -Name "Sales" -DisplayName "Sales"
Add-MailboxPermission -Identity "Sales" -User jmiller -AccessRights FullAccess
Add-RecipientPermission -Identity "Sales" -Trustee jmiller -AccessRights SendAs
Add-DistributionGroupMember -Identity "Client Services" -Member jmiller
Set-Mailbox -Identity jmiller -ProhibitSendReceiveQuota 200GB
```

## Lab Status

All 8 planned tickets are complete, covering mailbox and calendar permissions, distribution list mail flow, message tracing, quarantine handling, transport rules, onboarding provisioning, and automatic replies.
