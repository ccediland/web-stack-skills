---
title: Legal compliance — the aviso de privacidad template and the MX posture
summary: TEMPLATE, NOT LEGAL ADVICE — the aviso structure under the new 2025 LFPDPPP (verified against the DOF decree), the short-notice-at-collection pattern, the no-cookie-banner posture, what changed in required content, and the review-gates (missing reglamento, September 2026 reform) — added in W4.
last_updated: 2026-08-18
applies_to: es-MX sites collecting personal data (forms, analytics) · LFPDPPP published DOF 2025-03-20, in force 2025-03-21 · oversight = Secretaría de Anticorrupción y Buen Gobierno
---

# Legal compliance — aviso de privacidad (MX)

> Everything here is a STRUCTURAL TEMPLATE with slots — never legal advice, never final copy. Legal content is owner-and-lawyer-gated (the catalog's standing rule since the forms recipe), and every generated aviso ships visibly marked as a placeholder until ratified. What this recipe contributes is the verified legal FRAME (which law, which authority, which content the notice must carry) so the slots are the right slots.

## Contents

- The verified frame
- What the new law changed in the aviso
- The template (aviso integral)
- The short notice at collection
- Cookies and the banner question
- What stays out of scope
- Review-gates

## The verified frame

- The LAW is new: LFPDPPP published in the DOF 2025-03-20, in force 2025-03-21, replacing the 2010 law. INAI was dissolved; oversight now sits with the Secretaría de Anticorrupción y Buen Gobierno (SABG).
- The REGLAMENTO is not: the decree's transitory articles set a 90-day deadline for harmonized regulations (expired June 2025) that remains unmet; the 2011 reglamento and the 2013 Lineamientos del Aviso de Privacidad were NOT on the decree's abrogation list and apply suppletorily where they do not contradict the new law. Where they conflict, the 2025 law wins.
- No grace period exists for private companies — obligations applied from day one, and the SABG enforces actively (a 42.8M MXN fine over biometric-data violations landed in July 2026). "The reglamento is missing" is not a compliance pause.
- Consent model: tácito (implied) consent remains valid for non-sensitive data; express consent for sensitive/financial data. The aviso is mandatory and must be available at the moment of collection.

## What the new law changed in the aviso

Content the template below reflects, versus pre-2025 avisos:

- Finalidades must be EXHAUSTIVE and concrete — catch-all language ("entre otros", "among others") no longer passes.
- Automated decision-making: if decisions with legal effects are made about the person by automated means (including AI), the aviso must say so. A lead form with no automated decisioning states nothing; adding scoring or automated filtering later CHANGES the aviso.
- Data-retention periods: state them (defined periods or the criteria that determine them).
- Transfers moved out of the mandatory aviso content into their own regime (Art. 35 of the new law) — the template keeps a transfers slot because disclosure remains good practice and the old lineamientos required it, but its legal footing changed.

## The template (aviso integral)

Every {SLOT} is owner/lawyer content; the structure is the recipe's contribution. Ship with a visible placeholder badge until ratified.

```text
AVISO DE PRIVACIDAD INTEGRAL — {NOMBRE COMERCIAL}

Responsable. {RAZÓN SOCIAL}, con domicilio en {DOMICILIO COMPLETO},
es responsable del tratamiento de sus datos personales.

Datos que recabamos. A través de {el formulario de contacto / WhatsApp /
llamadas}: nombre, medio de contacto (correo o teléfono) y el mensaje que
usted nos comparte. {AJUSTAR según campos reales del form.}
No recabamos datos sensibles.

Finalidades. Sus datos se utilizan para: (1) responder a su solicitud de
contacto; (2) dar seguimiento a la contratación del servicio solicitado;
(3) {FINALIDAD ADICIONAL CONCRETA — sin "entre otros"}.

Medición del sitio. Este sitio utiliza analítica cookieless ({HERRAMIENTA,
p.ej. Umami}) que mide páginas visitadas, clics de contacto y país de
origen sin cookies y sin identificar al visitante.

Decisiones automatizadas. {SI NO APLICA, OMITIR SECCIÓN / Si aplica:
descripción de la decisión automatizada y su lógica general.}

Plazo de conservación. Sus datos de contacto se conservan durante
{PLAZO O CRITERIO, p.ej. "el tiempo necesario para atender su solicitud
y hasta N meses después"} y posteriormente se suprimen.

Derechos ARCO. Usted puede ejercer sus derechos de acceso, rectificación,
cancelación y oposición, así como revocar su consentimiento, escribiendo a
{CORREO DEL RESPONSABLE}. Responderemos en los plazos que marca la ley.

Transferencias. {NO transferimos sus datos a terceros / detalle de
transferencias y su fundamento.}

Cambios al aviso. Cualquier modificación se publicará en esta página.

Última actualización: {FECHA}.
```

## The short notice at collection

The form itself carries the SHORT notice — this stack already ships it via the forms recipe's consent checkbox: identity of the responsable, primary finalidad, and a link to the aviso integral. The pattern is one sentence plus the link, adjacent to the consent checkbox, never buried below the submit button. The analytics paragraph lives in the integral, not the short notice.

## Cookies and the banner question

Mexico has NO cookie-banner mandate. The catalog line, standing since the analytics recipe: cookieless elimina el banner, no el aviso. With the cookieless default (Umami, CF Web Analytics) no consent UI ships and the analytics disclosure lives in the aviso. The posture CHANGES if identifying tech enters — PostHog with replay, GA4, remarketing pixels — then consent gating and an expanded aviso are required, and that is a per-project legal conversation, not this template.

## What stays out of scope

- Términos y condiciones, contracts, warranties — pure lawyer territory; the recipe does not even template them.
- Sector-specific regimes (health data, minors, financial) — anything touching sensitive data leaves template land immediately.
- Non-MX audiences: material EU traffic brings GDPR logic (different consent model, different documents) — different sheet entirely.

## Review-gates

| Watch | Today (2026-08-18) | Flip |
|---|---|---|
| Reglamento of the 2025 law | unpublished, ~14 months past deadline; 2011/2013 instruments suppletory | publication → re-verify the template's required-content list |
| Reform initiative | SABG-led reform headed to Congress ~September 2026 (right-to-delisting, credit bureaus, self-correction mechanism discussed) | passage → re-run this recipe's frame |
| SABG guidance | no cookie/tracking-specific guidance published | any published criteria → adopt over inference |
| Enforcement climate | active (FMF fine, July 2026) | escalation raises the cost of stale avisos — keep the ratification gate honest |
