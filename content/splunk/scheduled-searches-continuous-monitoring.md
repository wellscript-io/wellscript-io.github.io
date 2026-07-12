---
title: "Scheduled Searches and Alerts That Satisfy Continuous Monitoring"
date: 2026-07-11
draft: false
tags: ["Splunk", "Continuous Monitoring", "RMF", "SPL", "Federal Cybersecurity"]
description: "How scheduled searches and alerts in Splunk turn continuous monitoring from a policy requirement into evidence an assessor will accept."
cover:
  image: "/images/scheduled-searches-continuous-monitoring-cover.png"
  alt: "Scheduled Searches and Alerts That Satisfy Continuous Monitoring"
  relative: false
  hidden: true
---

*New to how Splunk fits into compliance work? Start with [How Splunk Supports Cybersecurity Evidence](/splunk/how-splunk-supports-cybersecurity-evidence/).*

Most people in federal IT have written "continuous monitoring" into a security plan before they have actually built anything that monitors continuously.

*So what turns a ConMon strategy from a paragraph in a document into something an assessor can verify, and what does Splunk actually need to be doing for that to hold up?*

## The Gap Between the Policy and the Platform

NIST SP 800-137 describes an Information Security Continuous Monitoring program as an ongoing awareness of risk, vulnerabilities, and threats. DoDI 8510.01 expects that awareness to feed the authorization decision on a continuing basis, not just at the three-year reassessment. The document usually gets written first. The searches that are supposed to produce that awareness get written later, if at all, and often only after an assessor asks to see one running.

A dashboard people glance at during business hours is not continuous monitoring. It is a snapshot someone happened to look at. Continuous monitoring means something is checking on a schedule whether anyone is watching or not, and it means there is a record of that check having happened -- not just a record of what it found.

## Scheduled Searches Are the Mechanism, Not the Report

A scheduled search in Splunk runs an SPL query on a defined interval and does something with the result -- writes it to a summary index, triggers an alert, or populates a dashboard panel on refresh. The interval is the part that makes it continuous monitoring rather than ad hoc review. A search that runs every fifteen minutes against authentication failures is producing evidence on a cadence. A search someone runs manually once a week when they remember to is producing a data pull.

> **SysAdmin translation:** A scheduled search covering failed privileged logon attempts, saved against `index=wineventlog EventCode=4625 Logon_Type=...`, running every 15 minutes with results written to a summary index, is direct evidence for AC-2(12) account monitoring for atypical use and AU-6 audit review. The saved search's execution history in `_internal` -- not the dashboard -- is what proves the check actually ran on schedule.

**The search that never fires is not evidence of nothing happening.** It is evidence of nothing being checked. Assessors who understand SIEM tooling know the difference between "we had zero findings" and "we have no record the search ever ran." The saved search's job history, viewable under Settings > Searches, Reports, and Alerts, shows run count and last run time. That history is the artifact. Screenshot it, or better, export it, alongside the search itself.

**Alerts turn a scheduled search into something that reaches a person.** An alert fires when a scheduled search's results cross a defined condition -- more than zero results, a count above a threshold, a specific field value present. What the alert does next matters for ConMon evidence: an email notification with no other action leaves a trail in an inbox that is easy to lose. Writing the alert action to a summary index, or triggering a ticket in whatever the SOC uses for tracking, keeps the record inside the system rather than dependent on someone's mailbox retention policy.

## What Counts as Evidence Versus What Counts as Noise

**A threshold set too low drowns the signal the search exists to catch.** An alert configured to fire on any failed login will fire constantly in a normal environment and will get muted, disabled, or ignored within a month -- at which point the control it was supporting is not actually being monitored, regardless of what the SSP says. Alert thresholds tuned against a baseline of normal volume, and revisited when the environment changes, are part of what makes the monitoring credible rather than theoretical.

**A search that only counts is different from a search that correlates.** Counting failed logons tells you volume. Correlating failed logons against source IP, time of day, and account type tells you whether the volume is expected. The second version is what actually supports AC-2 and SI-4 in a way that survives a follow-up question about why a threshold was set where it was.

**The POA&M and the alert configuration should point at each other.** If a POA&M references continuous monitoring of a specific control as compensating evidence, the scheduled search enforcing that monitoring needs to actually exist, actually be running, and actually be traceable back to that control number. An SSP that claims ConMon coverage the SIEM configuration cannot back up is one of the more common gaps assessors flag, because it is one of the easiest gaps to check.

## The Practical Takeaway

Continuous monitoring becomes real the moment a scheduled search is running on an interval, producing a traceable record of its own execution, and tied to a specific control rather than a general sense of watching the network.

**Build the search before writing the sentence.** If the SSP says a control is continuously monitored, there should be a saved search with that control number in its description field before the sentence is finalized, not after.

**Treat run history as the artifact, not the results.** A clean result with no execution record proves nothing. Export or screenshot the job history alongside any results you are presenting as evidence.

**Tune thresholds against a real baseline.** An alert nobody can tolerate gets disabled quietly, and a disabled alert is a control that stopped being monitored without anyone updating the paperwork.

**Map every ConMon-referencing POA&M entry back to a specific saved search.** If you cannot point to the search, the POA&M is describing an intention, not a control.

Get that mapping right once and the next assessment conversation about continuous monitoring is a walkthrough of what already exists, not a scramble to build it.
