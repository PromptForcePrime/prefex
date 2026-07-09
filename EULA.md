# Prefex End User License Agreement (EULA)

**Version 1.0 — Effective July 8, 2026**

Copyright © 2026 DayTwoAI LLC. All rights reserved.

This End User License Agreement ("**Agreement**") is a binding legal agreement
between you, either an individual or the entity you represent ("**You**" or
"**Licensee**"), and DayTwoAI LLC, a Texas limited liability company doing
business as Promptforce.ai ("**Licensor**," "**we**," "**us**"),
governing your use of the **Prefex** software — the Prefex proxy binary, its
command-line interface, dashboard, bundled model/encoder assets, configuration
tooling, documentation, and any updates we make available (collectively, the
"**Software**").

**By downloading, installing, copying, running, or otherwise using the Software,
you agree to be bound by this Agreement. If you do not agree, do not download,
install, or use the Software.** If you are entering into this Agreement on behalf
of an organization, you represent that you have authority to bind that
organization, and "You" refers to that organization.

---

## 1. Definitions

- **"Proxy"** means the self-hosted Prefex server process that receives requests
  from your client tools and forwards them to an Upstream Provider.
- **"Upstream Provider"** means any third-party large-language-model or API
  service to which the Software forwards your requests, including without
  limitation Anthropic, OpenAI, and Google.
- **"Client Tool"** means any software you point at the Proxy (for example Claude
  Code, Codex, Cursor, Aider, Zed, or your own application).
- **"Free Tier"** means any version, feature set, or evaluation use of the
  Software made available at no charge.
- **"Paid Tier"** means any feature set, seat, or capability unlocked by a valid
  license key we issue in exchange for payment (including team features).
- **"License Key"** means the credential we issue that enables a Paid Tier.
- **"Local Data"** means data the Software stores on infrastructure you control,
  as described in Section 10.

---

## 2. License Grant

Subject to your continuous compliance with this Agreement (and, for Paid Tier
features, payment of applicable fees), Licensor grants you a limited,
non-exclusive, non-transferable, non-sublicensable, revocable license to:

(a) download, install, and run the Software **in object-code (binary) form** on
    machines or servers you own or control, for your own personal use or your
    internal business purposes;

(b) use the Software as a proxy to route, cache, compress, and instrument
    requests between your Client Tools and Upstream Providers whose services you
    are authorized to use; and

(c) use the number of seats, developers, or installations permitted by your then-
    current Tier and License Key.

The Free Tier is licensed for evaluation and ordinary use "as is" and may be
changed or discontinued at any time. All rights not expressly granted are
reserved by Licensor.

---

## 3. License Keys, Tiers, and Payment

(a) Paid Tier features are enabled by a valid License Key. License verification
    is performed **locally and offline**; the Software does not phone home to
    activate a key.

(b) A License Key is bound to a locally generated install identifier and to the
    seat or usage limits stated when it was issued. You may not share, resell,
    publish, or circumvent a License Key, or use the Software beyond the seats or
    scope your key authorizes.

(c) Fees, billing, renewal, and refund terms (if any) for a Paid Tier are
    governed by the ordering page, invoice, or separate Terms of Service under
    which you purchased. Non-payment, chargeback, or expiry of a Paid Tier
    terminates the corresponding Paid Tier license grant; the Software may revert
    to Free Tier behavior or disable Paid Tier features.

---

## 4. Restrictions

You may **not**, and may not permit any third party to:

(a) reverse engineer, decompile, disassemble, deobfuscate, or otherwise attempt
    to derive the source code, structure, or algorithms of the Software, except
    to the limited extent this restriction is expressly prohibited by applicable
    law (and then only after giving Licensor written notice and an opportunity to
    provide interoperability information);

(b) modify, adapt, translate, or create derivative works of the Software, or
    tamper with, bypass, or defeat its license enforcement, obfuscation,
    metering, or integrity mechanisms;

(c) redistribute, publish, sublicense, rent, lease, lend, sell, host as a
    service, or otherwise transfer the Software or any License Key to any third
    party;

(d) remove, alter, or obscure any copyright, trademark, or other proprietary
    notice in or on the Software;

(e) use the Software to develop, train, or improve a product or service that
    competes with the Software, or to benchmark it for that purpose, without our
    prior written consent;

(f) use the Software to violate, or to cause you to violate, the terms of service,
    acceptable-use policy, or rate limits of any Upstream Provider (see Section
    5); or

(g) use the Software in violation of any applicable law, regulation, or third-
    party right.

---

## 5. Upstream Providers and Your Responsibilities

**The Software is a conduit to third-party services that we do not operate or
control.** You acknowledge and agree that:

(a) Your use of any Upstream Provider through the Software is governed by **that
    provider's own terms of service, acceptable-use policy, and pricing**, and
    you are solely responsible for reviewing and complying with them, including
    any terms governing automated access, proxying, credential use, or OAuth
    tokens.

(b) You are solely responsible for the API keys, OAuth tokens, subscriptions, and
    accounts you use with the Software, for all activity conducted through them,
    and for **all fees, token charges, and usage costs** billed by Upstream
    Providers, including any charges resulting from routing, caching, prompt
    replay, retries, warming, or other Software behavior.

(c) Prefex is an **independent product and is not affiliated with, sponsored by,
    or endorsed by** Anthropic, OpenAI, Google, or any other Upstream Provider or
    Client Tool vendor. Product names are used for identification only and belong
    to their respective owners.

(d) Any cost-savings, routing, compression, latency, or cache figures reported by
    the Software are **estimates for informational purposes only**, are not a
    guarantee, and actual results depend on your workload, configuration, and
    provider pricing.

---

## 6. Local System Modifications

To function, the Software may read and modify configuration on machines you
control — for example, patching a Client Tool's settings (such as
`~/.claude/settings.json`) to point at the Proxy, forwarding OAuth bearer tokens,
and writing files under `~/.prefex/`. You authorize these actions. You are
responsible for maintaining backups of your configuration. **Licensor is not
liable for interruption of Client Tool sessions, credential re-authentication,
token invalidation, or configuration changes** that result from installing,
running, stopping, or uninstalling the Software.

---

## 7. Ownership and Intellectual Property

The Software is **licensed, not sold**. Licensor and its licensors retain all
right, title, and interest in and to the Software, including all copyrights,
trademarks, patents, trade secrets, and other intellectual property rights. No
rights are granted to you except as expressly stated in this Agreement. If you
provide feedback or suggestions, you grant Licensor a perpetual, irrevocable,
royalty-free license to use them without restriction.

---

## 8. Data, Privacy, and Telemetry

The Software is **self-hosted and runs on infrastructure you control.** We do not
operate a cloud service in your request path. Specifically:

(a) **Request content flows to the Upstream Provider you configure, not to us.**
    Your prompts and responses travel from your Client Tool, through your Proxy,
    to the Upstream Provider. That content is handled under the Upstream
    Provider's terms and privacy policy.

(b) **Local Data stays on your infrastructure.** By default the Software stores
    request metadata (token counts, model, latency, cost, cache indicators,
    hashes) in a local database, and — depending on your configuration — may
    additionally retain locally: originals of lossily compacted tool outputs (to
    support on-demand recovery, for a configurable retention period), extracted
    session-memory facts, an opt-in advisor training corpus, your configuration,
    and your License Key metadata. You control, can inspect, and can delete this
    Local Data. See the accompanying `TRUST.md` for specifics and verification
    steps.

(c) **No automatic upload.** The Software sends nothing to us automatically.
    Limited outbound connections to Licensor-associated endpoints occur only on
    explicitly opt-in paths — for example an optional catalog fetch when you
    enable telemetry, and a leaderboard submission that occurs **only when you
    initiate it** and that includes aggregate statistics plus the username and
    email you choose to enter. License verification requires no network call.

(d) **Optional protective features (such as PII scrubbing and the credential
    vault) are opt-in and provided "as is."** You are responsible for enabling
    and configuring them, and for reviewing what data your Client Tools send
    through the Proxy before it is forwarded upstream. The Software does not
    guarantee removal of sensitive data.

You are responsible for your own compliance with laws applicable to the data you
process through the Software (including data-protection and confidentiality
obligations to third parties).

---

## 9. Updates

Licensor may, but is not obligated to, provide updates, upgrades, bug fixes, or
new versions of the Software. When provided, they are part of the Software and
subject to this Agreement unless accompanied by separate terms. Licensor may
modify or discontinue any feature (including Free Tier features) at any time.

---

## 10. Open-Source and Third-Party Components

The Software includes or depends on third-party and open-source components, each
governed by its own license. Those licenses are listed in the
`THIRD_PARTY_LICENSES.md` and `NOTICE` files distributed with the Software. To
the extent any third-party license conflicts with this Agreement with respect to
that component, the third-party license governs that component.

---

## 11. Disclaimer of Warranties

THE SOFTWARE IS PROVIDED "**AS IS**" AND "**AS AVAILABLE**," WITHOUT WARRANTY OF
ANY KIND, WHETHER EXPRESS, IMPLIED, OR STATUTORY, INCLUDING WITHOUT LIMITATION
THE IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE,
TITLE, AND NON-INFRINGEMENT. LICENSOR DOES NOT WARRANT THAT THE SOFTWARE WILL BE
ERROR-FREE OR UNINTERRUPTED, THAT DEFECTS WILL BE CORRECTED, THAT IT WILL ACHIEVE
ANY PARTICULAR COST SAVING, ROUTING OUTCOME, LATENCY, OR CACHE-HIT RATE, OR THAT
IT WILL BE COMPATIBLE WITH ANY PARTICULAR CLIENT TOOL OR UPSTREAM PROVIDER, WHICH
MAY CHANGE AT ANY TIME. YOU ASSUME THE ENTIRE RISK ARISING OUT OF YOUR USE OF THE
SOFTWARE. Some jurisdictions do not allow the exclusion of implied warranties, so
some of the above may not apply to you.

---

## 12. Limitation of Liability

TO THE MAXIMUM EXTENT PERMITTED BY APPLICABLE LAW:

(a) IN NO EVENT WILL LICENSOR BE LIABLE FOR ANY INDIRECT, INCIDENTAL, SPECIAL,
    CONSEQUENTIAL, EXEMPLARY, OR PUNITIVE DAMAGES, OR FOR ANY LOSS OF PROFITS,
    REVENUE, DATA, GOODWILL, OR BUSINESS, OR FOR **ANY FEES, TOKEN CHARGES, OR
    OTHER COSTS BILLED BY AN UPSTREAM PROVIDER**, OR FOR ANY SUSPENSION,
    TERMINATION, OR OTHER ACTION TAKEN BY AN UPSTREAM PROVIDER AGAINST YOUR
    ACCOUNT, ARISING OUT OF OR RELATING TO THE SOFTWARE OR THIS AGREEMENT, EVEN IF
    ADVISED OF THE POSSIBILITY OF SUCH DAMAGES AND EVEN IF A REMEDY FAILS OF ITS
    ESSENTIAL PURPOSE.

(b) LICENSOR'S TOTAL AGGREGATE LIABILITY FOR ALL CLAIMS ARISING OUT OF OR RELATING
    TO THE SOFTWARE OR THIS AGREEMENT WILL NOT EXCEED THE GREATER OF (i) THE
    AMOUNTS YOU ACTUALLY PAID LICENSOR FOR THE SOFTWARE IN THE TWELVE (12) MONTHS
    PRECEDING THE EVENT GIVING RISE TO THE CLAIM, OR (ii) **USD $50**.

(c) THE FOREGOING LIMITATIONS APPLY TO THE FULLEST EXTENT PERMITTED BY LAW. Some
    jurisdictions do not allow certain limitations, so some of the above may not
    apply to you.

---

## 13. Indemnification

You will defend, indemnify, and hold harmless Licensor and its officers,
employees, and agents from and against any claims, damages, liabilities, costs,
and expenses (including reasonable attorneys' fees) arising out of or relating to:
(a) your use of the Software; (b) your breach of this Agreement; (c) your
violation of any Upstream Provider's terms or any applicable law; or (d) the data
or content you process through the Software.

---

## 14. Term and Termination

(a) This Agreement is effective until terminated. It terminates automatically if
    you breach any term.

(b) Licensor may suspend or terminate the license (including any Paid Tier) for
    breach, non-payment, or as required by law.

(c) Upon termination, you must stop all use of the Software and destroy all copies
    in your possession or control. You may delete your Local Data at any time as
    described in `TRUST.md`.

(d) Sections 4, 5, 7, 8, and 11 through 18 survive termination.

---

## 15. Compliance with Laws; Export

You represent that you are not located in, and will not use or export the Software
in violation of, any applicable export-control or sanctions law, and that you are
not on any government restricted-party list. You are responsible for compliance
with all laws applicable to your use of the Software.

---

## 16. Governing Law and Dispute Resolution

This Agreement is governed by the laws of the State of Texas, USA, without regard
to its conflict-of-laws rules. The parties consent to the exclusive jurisdiction
and venue of the state and federal courts located in Texas, USA for any dispute
not subject to arbitration. **[OPTIONAL — consult counsel: insert a binding
arbitration clause and class-action waiver here if desired, and specify a Texas
county for venue, e.g., Travis County.]** The U.N. Convention on Contracts for the
International Sale of Goods does not apply.

---

## 17. Changes to this Agreement

Licensor may update this Agreement for future versions or releases of the
Software. The version accompanying a given release governs your use of that
release. Your continued use of the Software after an updated Agreement takes
effect for a release constitutes acceptance of the updated terms for that release.

---

## 18. General

(a) **Entire Agreement.** This Agreement (together with any ordering document or
    separate Terms of Service you accepted for a Paid Tier) is the entire
    agreement between you and Licensor regarding the Software and supersedes all
    prior or contemporaneous understandings.

(b) **Severability.** If any provision is held unenforceable, the remaining
    provisions remain in effect and the unenforceable provision is limited or
    reformed to the minimum extent necessary.

(c) **No Waiver.** Failure to enforce a provision is not a waiver of it.

(d) **Assignment.** You may not assign this Agreement without Licensor's prior
    written consent; any attempted assignment in violation is void. Licensor may
    assign it freely.

(e) **Relationship.** The parties are independent contractors; this Agreement
    creates no partnership, agency, or employment relationship.

(f) **U.S. Government Rights.** The Software is "commercial computer software"
    provided with only the rights granted to the public under this Agreement.

---

## 19. Contact

Questions about this Agreement: **contact@daytwoai.com** — DayTwoAI LLC d/b/a
Promptforce.ai.
