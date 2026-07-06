# 01 — Project Charter

## Purpose

Ironbridge Health Alliance is a flagship portfolio project designed to demonstrate enterprise-grade security architecture, blue team, and GRC skills in a healthcare context — one of the most complex and highest-stakes environments in the industry. It is built specifically to generate credible conversations with healthcare IT professionals, general IT hiring managers, and cybersecurity teams, in support of a job search for a Blue Team Security Architect role.

## Why healthcare

Healthcare environments combine traditional corporate IT, clinical systems, legacy infrastructure, medical IoT, complex identity federation (often across acquired entities), and some of the heaviest regulatory compliance requirements of any industry (HIPAA, HITECH). It is also the sector behind many of the highest-profile ransomware incidents of the past decade, which makes it rich, realistic territory for both defensive design and incident-pattern storytelling.

## What this is not

This project does not use or claim to use real, licensed clinical software (e.g., Epic, Cerner). Clinical systems (EHR, imaging, pharmacy) are represented by realistic, appropriately segmented generic application stand-ins — an intentional and disclosed design decision, not a shortcut.

## Success criteria

- A fully built, multi-site AD environment with a genuine cross-forest trust relationship
- Documented detection coverage (SIEM) across all sites
- A completed risk register moving from "open" to "mitigated" status through the build
- At least one full "vulnerable state → attack → redesign" narrative arc, documented end to end
- A published set of technical articles/posts tied to build milestones
- Direct engagement (comments, conversations, DMs) from healthcare IT, general IT, or cybersecurity professionals as a result of the content

## Scope

Multi-site healthcare system: Corporate HQ, one hospital, one acquired regional clinic — see documents 02 and 03 for architecture detail.

## Constraints

Single physical host: Dell OptiPlex 3040, 16GB RAM, 500GB HDD, KVM/QEMU hypervisor, Kali Linux as bare-metal attacker platform. All design decisions in this document set are made to fit this constraint, with a documented fallback plan if further trimming is required (see document 04).

## Brand

This project is documented and published under the existing Hack the Homelab brand (LinkedIn, Beehiiv, GitHub, YouTube) in the established Security Architect voice — calm, analytical, peer-to-peer, outcome-led.
