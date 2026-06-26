---
title: "How ACAS Findings Become Remediation Evidence"
date: 2026-06-26
draft: false
tags: ["ACAS", "RMF", "Vulnerability Management", "Federal Cybersecurity", "DoD Compliance"]
description: "ACAS findings do not become remediation evidence on their own. Here is how scan data moves from Tenable.sc into the documentation your AO actually reviews."
cover:
  image: "/images/how-acas-findings-become-remediation-evidence-cover.png"
  alt: "How ACAS Findings Become Remediation Evidence"
  relative: false
  hidden: true
---

*New to RMF? Start with [What Is RMF? A Practical Introduction for SysAdmins](/rmf/what-is-rmf/).*

Most people working in federal IT environments have pulled an ACAS report before they understood what that report was actually supposed to prove. The scan runs, Tenable.sc produces a list of findings sorted by severity, and somewhere in the process that output is supposed to turn into evidence that satisfies an AO. What nobody explains upfront is what happens between the scan finishing and a finding being considered remediated inside the RMF package.

*So what does ACAS actually produce, and how does that output become the remediation evidence your AO is looking for?*

## A Finding Is Not Proof of Anything Yet

When ACAS completes a credentialed scan, Tenable.sc generates plugin results for every host in scope. Each plugin fires against a condition -- an unpatched binary, a misconfigured registry key, a deprecated cipher suite -- and returns a result with a severity rating and plugin output text. That data sits in the repository. It has not done anything compliance-wise yet.

The distinction that matters here: a finding is an observation. Evidence is a documented response to that observation. ACAS produces observations at scale and with specificity. What you do with those observations determines whether they become evidence.

The plugin output itself is the raw material. Plugin 19506 -- Nessus Scan Information -- confirms that a credentialed scan ran successfully on a given host. Plugin 21745 -- OS Security Patch Assessment Failed -- tells you local checks did not complete, which means the findings for that host cannot be trusted as comprehensive. Those two plugins alone tell you whether a given host's data is worth anything in a compliance conversation. A scan where 21745 fired on 30% of your hosts is not evidence that your environment is patched. It is evidence that your credential configuration needs work.

> **SysAdmin translation:** Before you export anything as evidence, filter your Tenable.sc results by asset list. The "Good, Local Checks Enabled" dynamic asset list (logic: plugin 117887 AND NOT 21745 AND NOT 110385) tells you which hosts produced credentialed, authenticated scan data. Only those hosts' findings mean what the finding says they mean. Everything else needs a separate explanation.

## What CMRS and eMASS Are Actually Looking For

Tenable.sc publishes vulnerability data to CMRS -- the Continuous Monitoring and Risk Scoring system -- where JFHQ-DODIN and CYBERCOM track IAVM compliance across the enterprise. That publishing step is how your scan data enters the broader compliance picture. It is not optional, and it is not the same as uploading a PDF report to eMASS.

CMRS reflects your scan posture in near-real time. eMASS is where you build the static compliance record -- the one an AO or SCA reviews when assessing your authorization package. These are two different audiences with two different needs.

For eMASS, what counts as remediation evidence depends on what kind of finding you are closing. **Patch findings** require a rescan showing the plugin no longer fires against the patched host. The before-and-after scan data -- or a Tenable.sc report showing the finding resolved -- is the artifact you upload. **Configuration findings** from STIG audit files require the audit report output alongside whatever change documentation shows the control was set correctly. **Risk accepted findings** require a formal risk acceptance with AO signature, and the finding stays open in eMASS in a Risk Accepted status rather than closing.

The ACAS Best Practices Guide draws a clean line here: for TASKORD compliance, you need a PDF report at minimum that contains vulnerability results and host counts, failed credential rates, and OS list. That report is a point-in-time artifact. It demonstrates the scan happened, the scan was credentialed, and the data produced is trustworthy. Without that verification of scan quality, the remediation data the report contains has no standing.

## Severity Is Not Automatically Category

ACAS uses CVSS-based severity ratings -- Critical, High, Medium, Low, Informational. STIGs use a different system -- CAT I, CAT II, CAT III -- tied to the impact of the finding rather than its exploitability score. When ACAS runs STIG audit files, Tenable.sc repurposes its severity codes to map to benchmark compliance status: High severity means the benchmark check failed, Informational means it passed, and Medium typically means the check was undetermined -- often because the scan credential did not have sufficient privilege, or the system needed a reboot to reflect the current state.

**SCAP severity codes** used in compliance scan results map directly to CAT levels: High equals CAT I, Medium equals CAT II, Low equals CAT III. That mapping matters when you are writing a POA&M entry or documenting a finding in eMASS, because the wrong category creates problems during SCA review. A CAT I finding with no scheduled completion date and no interim mitigation is a conversation stopper in any authorization review.

**CVSS-based vulnerability findings** do not map to CAT levels the same way. A Critical CVSS score on a finding that cannot be exploited without local access in a classified environment is not automatically the same risk profile as the number implies. The ISSO and ISSM are responsible for that contextual analysis. ACAS surfaces the data. The humans determine the actual risk posture and document it accordingly.

## The Scan Workflow That Produces Usable Evidence

Running a vulnerability scan and calling the output evidence skips several steps the process actually requires. The scan has to be credentialed -- ACAS is explicit that unauthenticated scans miss the local checks that find the bulk of patch compliance findings. DISA's TASKORD 20-0020 requires credentialed authenticated scans of all active IP space. A scan that triggered plugin 21745 on a significant portion of your hosts did not meet that requirement, regardless of how many findings it produced.

Plugins are updated daily via the DISA plugin servers. TASKORD requires sites to update plugins daily, and the ACAS Best Practices Guide notes that IAVM data should be reflected in plugins within 48 hours of the IAVM announcement. If you ran a scan against stale plugins, your findings do not accurately reflect current IAVM exposure. That is not a technicality -- auditors checking CCRI readiness will ask.

The scan frequency matters too. CCRI inspections expect scans completed within a three-day window. Sites are responsible for ensuring their scanner-to-host ratio supports that cadence -- roughly one scanner per 2,000 to 4,000 hosts, depending on network topology and scan window requirements.

Once the scan data is in Tenable.sc and confirmed credentialed, the remediation loop looks like this: identify the finding, remediate the condition on the endpoint, rescan to confirm the plugin no longer fires, export the scan result or report showing the resolution, and upload that artifact to eMASS as the closure evidence for the corresponding POA&M entry or control implementation statement. The rescan is the step most people underestimate. Asserting that a patch was applied is not the same as showing ACAS did not find it anymore.

## The Practical Takeaway

ACAS findings become remediation evidence when the scan was credentialed, the data is current, the finding is accurately categorized, and the closure artifact shows the condition resolved.

**Confirm scan quality before you build anything around the output.** Check plugin 117887 and 21745 across your hosts before exporting anything as evidence. If your credentialed scan rate is low, the remediation data you are producing is not representative, and that gap will surface during any serious review.

**Know which system owns the finding.** CMRS tracks IAVM posture across the enterprise. eMASS holds the compliance record for your specific authorization boundary. A finding that is closed in your eMASS package but still showing open in CMRS is a discrepancy that needs an explanation. The ACAS Best Practices Guide recommends reviewing CMRS data after publishing to confirm the data in CMRS matches what Tenable.sc shows.

**Match your evidence to the finding type.** Patch findings close with rescan data. Configuration findings close with audit results and change documentation. Risk accepted findings do not close -- they stay open with a documented acceptance. Uploading the wrong artifact type does not close the finding. It creates an artifact that an SCA will ask about.

**Treat the scan as the start of the evidence chain, not the end of it.** ACAS does the observation work. The ISSO does the analysis, documentation, and follow-through. The AO reviews the chain. Every gap between those steps shows up during assessment.
