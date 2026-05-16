> ⚠️ **Pre-release placeholder.** The first public release is gated on the
> NGI Zero Commons Fund grant decision (applied 2026-05, decision
> expected Q3 2026). The README below describes the planned package +
> roadmap so grant reviewers, future contributors, and prospective
> consumers can evaluate the project.

A free + open-source Python toolkit for **Portuguese short-term-rental
regulatory compliance**. Drop-in primitives for RNAL number
canonicalisation, SIBA boletim generation per Lei 23/2007, per-município
tourist-tax calculation, AT fatura helpers, and parsing of inbound
booking-confirmation emails from Booking.com, Airbnb, and VRBO.

## Why this exists

Operating a short-term rental in Portugal requires interacting with at
least five regulatory portals (RNAL via Balcão do Empreendedor, SIBA
via SEF SIRJI, AT for tax declarations, INE for tourism statistics, and
~50 município portals for tourist tax). Every implementation of these
primitives — closed-source SaaS like Chekin, Conforme, Hospitable;
internal hospitality-group tooling; ad-hoc PT accountant scripts —
re-implements the same basic logic, often incorrectly. This package
provides a single tested, documented, versioned baseline that any
consumer can build on.

## Status

| Component | Status |
|---|---|
| Spec + design | ✅ Complete (extracted from production Conforme codebase) |
| Grant funding | ⏳ NGI Zero Commons Fund application submitted 2026-05; decision ~Q3 2026 |
| Initial release | 🚧 Targeted v0.1 alpha within 90 days of grant award |
| Stable release | 🚧 Targeted v1.0 within 9 months of grant award |
| Community contributions | 🚧 Open after v0.1 alpha |

## Planned modules (post-release)

```
conforme_pt_toolkit/
├── rnal/                  # RNAL number canonicalisation, validation, register lookup
├── siba/                  # SIBA boletim XML generation per Lei 23/2007 (BAL.XSD schema)
├── tourist_tax/           # Per-município calculator (Lisboa, Porto, Funchal, Cascais, Sintra)
├── at/                    # Autoridade Tributária fatura/recibo format helpers
├── inbound_email/         # Vision-only parser for Booking.com / Airbnb / VRBO / Direct
├── compliance/            # Per-country evaluator + renewal tracker abstractions
├── crypto/                # Envelope-encryption pattern for storing operator credentials
└── municipios/            # Data directory — one file per município, contributed by community
```

## Planned API (illustrative)

```python
from conforme_pt_toolkit.rnal import canonicalise, validate, lookup

# Canonicalise + validate
canonicalise("12345/al")           # → "12345/AL"
validate("12345/AL")               # → True
validate("12345")                  # → ValidationError("missing /AL suffix")

# Live register lookup (cached)
record = lookup("12345/AL")
record.title                       # → "Casa do Mar"
record.status                      # → "active"
record.capacity                    # → 4
record.município                   # → "Lisboa"

# SIBA boletim XML for a guest stay
from conforme_pt_toolkit.siba import boletim
xml = boletim.generate(
    property_=record,
    guests=[guest_a, guest_b],
    check_in=date(2026, 6, 1),
    check_out=date(2026, 6, 5),
)
# → bytes containing BAL.XSD-conformant XML, ready to POST to SIRJI

# Tourist tax for a stay in Lisboa
from conforme_pt_toolkit.tourist_tax import calculate
calculate("LX", nights=4, guest_count=2)   # → Decimal("8.00")  (2 × 4 × €1.00)

# Inbound email parsing
from conforme_pt_toolkit.inbound_email import parse
booking = parse(email_html, email_text, subject="Booking confirmation")
booking.guest_name                 # → "Maria Silva"
booking.check_in                   # → date(2026, 6, 1)
booking.source_ota                 # → "booking_com"
booking.confidence                 # → 0.95
```

## Licence

**MIT** (planned, not yet committed to). Commercial use, modification,
and distribution permitted with attribution.

## Governance

Project lead: [Curtis Smith](https://github.com/Niteowlpt). Once
contributions open (post-v0.1 alpha), the governance model will follow
[The Apache Way](https://www.apache.org/theapacheway/index.html)
lite-weight variant — public RFC for breaking changes, contributor
agreement for substantial PRs, quarterly maintenance cadence published
in the README.

The project is structurally independent of [Conforme](https://conforme.info)
(the closed-source SaaS that originally housed the code); Conforme
will consume the published package as a downstream user.

## Roadmap (post-grant)

**Phase 1 — Foundation (months 1-3)**
- Extract + harden RNAL, SIBA, AT modules from Conforme private codebase
- Publish 0.1 alpha to PyPI
- Documentation site (Sphinx + ReadTheDocs)

**Phase 2 — Coverage (months 4-6)**
- Tourist-tax data layer for Lisboa, Porto, Funchal, Cascais, Sintra
- Inbound email parser with regex fallbacks for Airbnb + VRBO
- 90%+ test coverage with synthetic regulation-realistic fixtures

**Phase 3 — Community (months 7-9)**
- Public contribution workflow for new municípios
- Outreach to 5-10 closed-source SaaS vendors offering the toolkit as a baseline
- Conference talk at APHORT (autumn 2026)
- v1.0 stable release

**Phase 4 — Ecosystem (post-grant)**
- Companion `conforme-es-toolkit` led by community contributor (NRUA + ES STR primitives)
- IT / FR / DE roadmap items reviewed quarterly

## Funding

Initial development is funded by the [NGI Zero Commons Fund](https://nlnet.nl/commonsfund/)
(grant application submitted 2026-05). Long-term maintenance is funded
by [Conforme](https://conforme.info) — the closed-source SaaS that
originally housed the code — as a direct beneficiary of the toolkit's
existence.

## How to follow updates

- ⭐ Star this repo (you'll see release notifications)
- Watch the [NGI Zero project page](https://nlnet.nl/news/) for grant decision news
- Subscribe to release announcements: GitHub Releases page once v0.1 ships

## How to NOT use this (yet)

This README is a pre-release placeholder. **There is no installable
package as of this commit.** Attempting `pip install conforme-pt-toolkit`
will fail. If you need PT regulatory primitives for a production system
today, your options are:

- Build the primitives yourself from the underlying regulations
  ([Lei 23/2007](https://dre.pt/dre/legislacao-consolidada/lei/2007-34514875),
  [DL 128/2014](https://dre.pt/dre/legislacao-consolidada/decreto-lei/2014-118975411))
- Consume a closed-source SaaS like [Conforme](https://conforme.info),
  [Chekin](https://chekin.com), or [Hospitable](https://hospitable.com)
- Wait for v0.1 alpha (target Q4 2026, gated on grant decision)
