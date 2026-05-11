---
title: "Security+ Foundational Terminology"
tags: ["security-plus", "risk-management", "security-fundamentals"]
prerequisites: []
status: "draft"
last_updated: "2025-10-01"
todos:
  - "Add risk calculation metrics (SLE, ARO, ALE)."
  - "Include concrete examples for each control category."
---

## Summary

Baseline Security+ vocabulary covering vulnerabilities, threats (adversarial and non-adversarial), threat actors, exploits, layers of defensive controls, and how frameworks tailor safeguards to organizational risk.

## Key Terms

-   **Vulnerability:** A weakness in a system, process, or human control that could be exploited.
-   **Threat:** A potential danger to the system when a vulnerability exists.
    -   **Adversarial Threat:** Originates from an intentional actor, such as a hacker, criminal syndicate, or insider.
    -   **Non-Adversarial Threat:** Stems from accidents or natural events, for example natural disasters or human error.
-   **Threat Actor:** The entity—human or automated—with the capability and intent to carry out a threat (e.g., hacker, malicious insider).
-   **Exploit:** The action a threat actor takes to successfully take advantage of a vulnerability.
-   **Countermeasure:** A specific control implemented to reduce risk by addressing a vulnerability or altering threat likelihood/impact.
-   **Defense in Depth:** A security strategy that layers multiple controls so the failure of one does not expose the asset.
-   **Cascade Effect:** When exploitation of one vulnerability enables additional vulnerabilities to be exploited.
-   **Diversity:** Applying different types or variants of controls to avoid single points of failure.
-   **Baseline:** The minimum level of security that must be met across systems or environments.
-   **Control Objective:** The specific security goal a control is intended to achieve.

## Controls

Controls are safeguards or countermeasures—technical, administrative, or physical—that reduce risk by changing either the likelihood or the impact of a threat exploiting a vulnerability.

1.  Reduce or eliminate the underlying vulnerability.
2.  Reduce or eliminate the likelihood of a threat exploiting that vulnerability.
3.  Mitigate the impact if the threat is realized.

### Control Evaluation Criteria

-   **Functionality:** Does the control perform the required security function?
-   **Effectiveness:** How well does the control reduce risk in practice?
-   **Assurance:** Confidence level that the control is designed, implemented, and operating as intended.

## Frameworks and Standards

-   **NIST SP 800-53:** Catalog of security controls for U.S. federal information systems.
-   **NIST Frameworks:** Cybersecurity Framework (CSF), Privacy Framework, and Risk Management Framework (RMF) that guide risk assessment and control selection.
-   **ISO/IEC 27014:2020:** International standard for governing information security programs.

## Tailoring Controls

-   **Scoping:** Eliminating controls that are not applicable to the organization or system under review.
-   **Tailoring:** Customizing applicable controls to align with the organization’s specific environment and needs.
-   **Compensating Controls:** Alternate safeguards that provide an equivalent level of protection when the primary control cannot be implemented.
-   **Supplemental Controls:** Additional safeguards added to exceed the baseline or address unique risks.
-   **Cost-Benefit Analysis:** Evaluating the financial cost versus the security benefit of implementing a given control.

## References

-   CompTIA Security+ course, Episode 1 (personal notes).
