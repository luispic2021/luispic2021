---
layout: page
title: Experience
eyebrow: Detailed profile
description: Product and technical leadership across fintech, risk decisioning, compliance technology, ML-powered detection, data products, and context engineering.
permalink: /cv/
---

<div class="cv-intro">
  <img src="{{ '/assets/images/headshot.jpg' | relative_url }}" alt="Portrait of Luis P. Perez" width="800" height="800">
  <p>I’m a technical product leader who has spent my career translating complex business and regulatory problems into products, platforms, and decision systems. My experience spans international payments, real-time KYC and risk, AML technology, ML-powered enforcement, and foundational identity data products for Data Science, ML Engineering, and Analytics.</p>
</div>

## Experience

<section class="cv-role" markdown="1">
  <h2>The Walt Disney Company</h2>
  <p class="cv-role-meta">Senior Product Manager · 2025—Present · San Francisco Bay Area</p>

  <h3>Identity Data Products · 2026—Present</h3>

  - Own product strategy and roadmap for reusable data products spanning identities, accounts, profiles, and devices.
  - Gather requirements from Data Science, ML Engineering, Analytics, and other downstream consumers, then sequence delivery with Data Engineering.
  - Use Databricks Unity Catalog, notebooks, SQL, and Python for data exploration and lightweight analysis.
  - Focus the roadmap on making core assets more logical, discoverable, understandable, and useful for analytics and model development.

  <h3>Paid Sharing Detection · 2025—2026</h3>

  - Set direction from heuristic-only enforcement toward ML-based decisioning, evaluating four models and helping productionize a conversion-propensity model.
  - Independently analyzed model-score volatility to evaluate the risk of inconsistent customer experiences; validated the findings with ML Engineering and helped unblock an experiment that led to production use.
  - Initiated a predictive experiment metric with Analytics that matured in roughly 3 weeks rather than 12, enabling decisions 75% faster.
  - Challenged aggregate global results when one market dominated the sample, then implemented region-specific experiments that revealed meaningful performance differences and avoided an unsupported global rollout.
</section>

<section class="cv-role" markdown="1">
  <h2>Block / Square</h2>
  <p class="cv-role-meta">Technical Program Manager, BSA / Compliance Technology · 2020—2025 · Remote</p>

  - Joined as the first Technical Program Manager while the Compliance Technology organization was being built.
  - Worked across Compliance, Legal, Product, Operations, and Engineering on customer monitoring, transaction monitoring, regulatory reporting, quality control, remediation, and multi-jurisdiction launch requirements.
  - During a LexisNexis PEP database migration, identified a persistent entity identifier, proposed a suppression configuration, and experimentally demonstrated approximately 90% improvement in avoiding resurfaced alerts while maintaining compliance requirements.
  - Spearheaded a case-management proof of concept that let Operations experience a purpose-built adverse-media workflow, creating alignment with Engineering around a modern modular platform.
</section>

<section class="cv-role" markdown="1">
  <h2>Beam Solutions / Jumio</h2>
  <p class="cv-role-meta">Head of Product, Beam → Senior Product Manager, Jumio · 2019—2020 · San Francisco</p>

  - Owned prioritization and roadmap for an AML technology startup, working as the sole PM alongside a product designer and a small product/engineering organization.
  - Partnered with the CTO to design a common financial data model organized around individuals, transactions, payment sources, and accounts.
  - Led a redesign that brought the core data entities into a coherent investigation view and replaced technical query syntax with guided construction for non-technical compliance users.
  - Established a formal Jira-based release and approval process aligned with SOC 2 change-management controls.
  - Continued at Jumio after its acquisition of Beam, focused on integrating transaction-monitoring capabilities into the broader suite.
</section>

<section class="cv-role" markdown="1">
  <h2>Xoom / PayPal</h2>
  <p class="cv-role-meta">Group Product Leadership · 2014—2019 · Guatemala and San Francisco</p>

  - Built and managed a distributed Risk Product organization of up to four PMs spanning real-time KYC, internal risk tools, bill-payment experiences, and platform modernization.
  - Hired and developed PMs, conducted performance reviews, promoted a PM into people management, and led teams across countries.
  - Helped transform a historically constrained $3,000 transfer experience through risk-based, progressive KYC and synchronous decisioning—allowing eligible customers to send higher values with friction applied according to risk and compliance needs.
  - Led product strategy at the intersection of customer risk scoring, transaction context, identity signals, rules, compliance thresholds, and customer experience.
  - Supported integration of BlueKite’s bill-payment capabilities and migration of parts of the legacy Send Money platform toward independently deployable services.
  - Enabled a customer self-service legal-name update flow that removed an unnecessary Customer Service dependency while preserving identity requirements.
</section>

<section class="cv-role" markdown="1">
  <h2>BlueKite</h2>
  <p class="cv-role-meta">Head of Integrations · Before acquisition by Xoom in 2014 · Guatemala</p>

  - Operated at the intersection of partnerships, product, and engineering for an international bill-payment fintech startup.
  - Read provider API documentation, participated in commercial and technical discovery, and converted disparate partner documentation into standardized implementation requirements.
  - Managed an engineer and helped the small team integrate payment and utility providers faster and more consistently.
</section>

## Independent technical project

### Automated trading and backtesting platform

A production-deployed personal system developed primarily in Python. It has included modular integrations with Schwab, Interactive Brokers, and Massive.io; automated execution; stop-loss and profit logic; pandas-based backtesting; trade logs and performance analysis; parameter experimentation; and a Random Forest prototype using engineered technical indicators.

The project follows a repeatable product-development loop: **hypothesis → backtest → experiment and iterate → productionize → monitor**.

[Read more at Code & Candlesticks](https://codeandcandlesticks.com) or [explore my GitHub](https://github.com/luispic2021).

## Technical depth

| Area | Proficiency |
| --- | --- |
| Python / pandas | Working: scripts, notebooks, analysis, backtesting, and AI-assisted code review |
| SQL | Intermediate: data exploration, joins, data models, and validated AI-assisted query development |
| APIs | Strong product and technical fluency: documentation, authentication patterns, consumption, and integration requirements |
| Databricks | Working product/data user: Unity Catalog, notebooks, SQL, and data exploration |
| Snowflake | Working product/data user |
| Datadog | Working: real-time product and experiment monitoring dashboards |
| ML products | Experiment design, model rollout decisions, stability analysis, precision/recall concepts, and customer-impact judgment |
| Context engineering | Certified grounding in retrieval, memory, tools/MCP, compression, and multi-agent or long-horizon architectures |

## Leadership

- Managed up to four direct Product Managers across local and distributed teams.
- Hired, developed, reviewed, and performance-managed PMs; promoted a PM into people management.
- Served as Head of Product with responsibility for a full company roadmap.
- Presented regularly to VP and C-level leaders and at company all-hands.
- Led across Product, Engineering, Compliance, Legal, Operations, Data Science, ML Engineering, Analytics, and Data Engineering.

## Education &amp; development

**B.S. Electrical Engineering**  
Universidad Galileo

<div class="cv-credential">
  <a href="{{ site.databricks_credential_url }}" target="_blank" rel="noopener noreferrer" aria-label="Verify Luis P. Perez's Databricks Certified Context Engineer Associate credential">
    <img src="{{ '/assets/images/databricks-context-engineer-badge.png' | relative_url }}" alt="Databricks Certified Context Engineer Associate badge" width="570" height="792" loading="lazy">
  </a>
  <p><strong>Databricks Certified Context Engineer Associate</strong><br>Earned July 2026 · Valid through July 2028<br><a href="{{ site.databricks_credential_url }}" target="_blank" rel="noopener noreferrer">Verify credential ↗</a> · <a href="https://www.databricks.com/learn/certification/context-engineer-associate">Certification overview</a></p>
</div>

## Connect

[LinkedIn](https://www.linkedin.com/in/luispic/) · [GitHub](https://github.com/luispic2021) · [Code & Candlesticks](https://codeandcandlesticks.com)
