# cloud-itonami-iso3166-jpn-mofa

Open ISO 3166 Agency Blueprint for **JPN-MOFA**: Ministry of Foreign Affairs
(外務省, MOFA) — a Japan-agency-level LEAF under
the `cloud-itonami-iso3166-jpn` country-level coordinator.

This repository designs a forkable OSS business for an independent
compliance consultant: an already-incorporated operator (typically one
already using `cloud-itonami-iso3166-jpn` for general Japan market entry)
gets a Compliance Advisor + independent **ODA Tender-Compliance Governor** to
navigate eligibility screening for Japan's Official Development Assistance (政府開発援助/ODA) tenders and JICA-funded international-development contract rules administered under MOFA's ODA policy framework.

## No robotics premise — digital/data service exemption

Agency-specific compliance navigation is a pure data/software service with
no physical-domain work — the same exemption class as `cloud-itonami-6310`
and `cloud-itonami-gtin-*`. `blueprint.edn` sets
`:itonami.blueprint/robotics false` and `:required-technologies` lists only
real capabilities (`:identity`, `:forms`, `:dmn`, `:bpmn`, `:audit-ledger`),
no `:robotics`.

## Core Contract

```text
operator intake + prior filing/compliance history
        |
        v
Compliance Advisor -> ODA Tender-Compliance Governor -> compliance draft, or human sign-off
        |
        v
gated filing / registration / compliance-program submission + audit ledger
```

No automated proposal can submit a filing or registration the governor
refuses, suppress a compliance record, or claim a legal conclusion the
governor has not cleared. `:filing/submit` is never in any phase's `:auto`
set — it always requires human sign-off (mirrors `cloud-itonami-M6910`'s
`filing-submit-never-auto-at-any-phase` invariant).

## What this is NOT

- **Not Ministry of Foreign Affairs (外務省) itself, and not the
  government of Japan.** See [`docs/business-model.md`](docs/business-model.md)
  for the boundary with `com-etzhayyim-ooyake`, `matsurigoto`,
  `com-etzhayyim-toritsugi`, `legal-entity.etzhayyim.com`,
  `cloud-itonami-M6910`, and the country-level `cloud-itonami-iso3166-jpn`.
- **Not legal or tax advice.** Every regulatory claim must cite the
  official MOFA source and route final filings to
  Japan-licensed counsel or a registered agent where the law requires
  licensed representation. The sources it may cite are enumerated in
  [`facts.edn`](facts.edn) — see below.

## Regulatory source register

[`facts.edn`](facts.edn) is the list of sources a claim in this repository is
allowed to cite: MOFA's ODA policy pages including the 開発協力大綱, the
statutes and orders behind MOFA, JICA and competitive tendering, and JICA's own
procurement rules. It is tx-data, so it loads like every other EDN corpus here:

```clojure
(d/transact conn (edn/read-string (slurp "facts.edn")))
```

A source not in the table has no spec-basis — extend the table, never invent an
id or a URL. Re-check every entry against the live authority with:

```bash
nbb scripts/verify-facts.cljk      # 0 = all verified, 1 = a source is wrong,
                                   # 2 = the run could not answer (not a pass)
```

Two things worth knowing before trusting a green run, both measured 2026-08-26
and both recorded in the file:

- **`laws.e-gov.go.jp` answers HTTP 200 for `/law/<anything>`**, including law
  ids that do not exist, and renders "not found" client-side. So the verifier
  never uses HTTP status for a statute — it resolves the id through the e-Gov
  law API and requires the title and law number to match. It proves that branch
  still discriminates, against an id that must not resolve, before it reports
  anything. A status-only check would have passed `322CO0000000165`, which this
  register briefly cited for 予算決算及び会計令 and which is not a law
  (the correct id is `322IO0000000165`; 勅令 is `IO`, not `CO`).
- **`www.mofa.go.jp` answers 403 to `curl`** over both HTTP/2 and HTTP/1.1, with
  or without a browser User-Agent, while answering 200 with full content to
  `fetch` at the same moment — the block is on the TLS fingerprint. A
  curl-based verifier would record this repository's own naming authority as
  permanently unreachable, and would look like it had measured that.

## Capability layer

Resolves via [`kotoba-lang/iso3166`](https://github.com/kotoba-lang/iso3166)
(code `JPN-MOFA`, `:parent "JPN"`, cross-referenced to ooyake's
`gov.jpn.mofa`). Required capabilities:

- :identity
- :forms
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
