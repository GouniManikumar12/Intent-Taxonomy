# Agentic Intent Taxonomy

Status: Public Draft  
Version: `0.1.0`  
Last updated: `2026-03-23`

## 1. Overview

The **Agentic Intent Taxonomy (AIT)** defines a standardized structure for representing **user intent inside AI-driven conversations**.

It is designed to:

- align with IAB Content Taxonomy
- enable monetization decisioning in conversational environments
- provide a consistent interface for AI platforms, exchanges, and advertisers
- support implementation across policy, auction, analytics, annotation, and model training systems

This repository contains:

- the normative overview in this README
- the JSON Schema in `agentic-intent-taxonomy.schema.json`
- canonical example envelopes in `examples/`

This public draft is ready for review, implementation feedback, and schema-level collaboration. Companion workstreams such as annotation guidelines and seed datasets are important, but they are not blockers for discussing or improving the specification itself.

## 2. Design Principles

### 2.1 Orthogonality

Each dimension represents a distinct signal:

- category != intent != policy != runtime

### 2.2 Industry Compatibility

- uses IAB taxonomy for content classification
- extends that layer with intent, policy, and monetization context

### 2.3 Real-Time Readiness

- designed for sub-500ms decision loops
- supports partial and degraded responses

### 2.4 Privacy-Aware

- no raw PII required
- supports hashed identity and minimal context

### 2.5 Auditability

- model prediction and system decision should be separable
- monetization decisions must be explainable after the fact

## 3. Core Dimensions

### 3.1 Content Classification (IAB-Aligned)

Canonical external reference:

- IAB Content Taxonomy 3.1 TSV: <https://github.com/InteractiveAdvertisingBureau/Taxonomies/blob/develop/Content%20Taxonomies/Content%20Taxonomy%203.1.tsv>

Compact transport form:

```json
{
  "iab_content": {
    "taxonomy_version": "3.1",
    "tier1": "Business and Finance",
    "tier2": "Business",
    "tier3": "Business I.T."
  }
}
```

#### 3.1.1 Recommended IAB Level Encoding

When a system needs stable matching, auditing, or downstream interoperability, the preferred representation is an explicit level object:

```json
{
  "iab_content": {
    "taxonomy": "IAB Content Taxonomy",
    "taxonomy_version": "3.1",
    "tier1": {
      "id": "52",
      "label": "Business and Finance"
    },
    "tier2": {
      "id": "53",
      "label": "Business"
    },
    "tier3": {
      "id": "72",
      "label": "Business I.T."
    },
    "mapping_mode": "nearest_equivalent",
    "mapping_confidence": 0.93
  }
}
```

Rules:

- must map to IAB taxonomy or an internal mapped equivalent
- `tier1` is required for monetization-aware classification
- `tier2` and `tier3` are strongly recommended for advertiser matching and reporting
- the canonical external reference for IAB mappings is the IAB Content Taxonomy 3.1 TSV: <https://github.com/InteractiveAdvertisingBureau/Taxonomies/blob/develop/Content%20Taxonomies/Content%20Taxonomy%203.1.tsv>
- if an exact IAB node is unavailable, systems should use the closest valid node and record `mapping_mode`
- internal extensions should never replace the IAB path; they should sit alongside it

### 3.2 Intent Classification

#### 3.2.1 Intent Type

```json
{
  "intent_type": [
    "informational",
    "exploratory",
    "commercial",
    "transactional",
    "support",
    "personal_reflection",
    "creative_generation",
    "chit_chat",
    "ambiguous",
    "prohibited"
  ]
}
```

#### 3.2.2 Intent Subtype (examples)

```json
{
  "commercial": [
    "product_discovery",
    "comparison",
    "evaluation",
    "deal_seeking",
    "local_search",
    "provider_selection"
  ],
  "transactional": [
    "signup",
    "purchase",
    "booking",
    "download",
    "contact_sales"
  ],
  "support": [
    "troubleshooting",
    "account_help",
    "billing_help"
  ]
}
```

#### 3.2.3 Decision Phase

Decision phase captures the user’s place in the decision journey. It is not the same thing as the interaction pattern.

```json
{
  "decision_phase": [
    "awareness",
    "research",
    "consideration",
    "decision",
    "action",
    "post_purchase",
    "support"
  ]
}
```

#### 3.2.4 Distinguishing Subtype vs Decision Phase

- `subtype` describes the interaction pattern, such as `comparison` or `product_discovery`
- `decision_phase` describes the funnel state, such as `research`, `consideration`, or `decision`

Example:

```json
{
  "intent": {
    "type": "commercial",
    "subtype": "comparison",
    "decision_phase": "decision"
  }
}
```

#### 3.2.5 Scores

```json
{
  "intent": {
    "confidence": 0.0,
    "commercial_score": 0.0
  }
}
```

Scoring guidance:

- `confidence` expresses classifier certainty in the assigned labels
- `commercial_score` expresses monetizable commercial relevance, not safety or policy clearance
- both scores use a `0.0` to `1.0` range
- `summary` may be included as an optional human-readable explanation of the classification
- `summary` is non-normative and systems must not depend on it for policy, matching, ranking, or monetization decisions

#### 3.2.6 Ambiguity and Fallback

Real systems must support incomplete or low-confidence interpretations.

```json
{
  "fallback": {
    "applied": true,
    "fallback_intent_type": "exploratory",
    "fallback_monetization_eligibility": "not_allowed",
    "reason": "confidence_below_threshold"
  }
}
```

Rules:

- if confidence falls below a system-defined threshold, the system should prefer fallback behavior over forced monetization
- `ambiguous` should be used when the utterance cannot be cleanly resolved without additional context
- fallback behavior must default to a safe monetization posture

### 3.3 Policy and Monetization Decision Boundary

```json
{
  "policy": {
    "monetization_eligibility": "allowed",
    "eligibility_reason": "commercial_score_above_threshold",
    "decision_basis": "score_threshold",
    "applied_thresholds": {
      "commercial_score_min": 0.7,
      "confidence_min": 0.6
    },
    "sensitivity": "low",
    "regulated_vertical": false
  }
}
```

Rules:

- policy decisions must be independent from commercial attractiveness
- high-value intent does not override safety restrictions
- systems should define explicit threshold logic for monetization eligibility
- policy overrides always take precedence over score-based monetization
- regulated verticals may be monetizable only under platform-specific controls

### 3.4 Contextual Signals

```json
{
  "context": {
    "entities": ["HubSpot", "Zoho"],
    "constraints": {
      "budget": "low",
      "company_size": "small_team"
    }
  }
}
```

Rules:

- context should capture decision-relevant signals, not full conversation state
- entities should be normalized when possible
- constraints should remain sparse and privacy-aware

### 3.5 Opportunity Classification

```json
{
  "opportunity": {
    "type": "comparison_slot",
    "strength": "high"
  }
}
```

Rules:

- opportunity is downstream-facing and should not be confused with intent type
- opportunity may be `none` even when commercial intent exists
- policy can suppress opportunity even when the model predicts high commercial value
- `strength` expresses the quality of the monetization moment, not the model confidence

Valid `strength` values:

- `low`
- `medium`
- `high`

### 3.6 Temporal Signals

Intent evolves across turns and should optionally be represented over time.

```json
{
  "intent_trajectory": [
    "research",
    "consideration",
    "decision"
  ]
}
```

Rules:

- trajectory is optional but strongly recommended for multi-turn systems
- trajectory should reflect ordered user-state progression, not raw message history

## 4. Canonical Decision Envelope

The canonical audited object should separate model prediction from system decision.

```json
{
  "model_output": {
    "classification": {
      "iab_content": {
        "taxonomy_version": "3.1",
        "tier1": "Business and Finance",
        "tier2": "Business",
        "tier3": "Business I.T."
      },
      "intent": {
        "type": "commercial",
        "subtype": "comparison",
        "decision_phase": "decision",
        "confidence": 0.87,
        "commercial_score": 0.92,
        "summary": "User is evaluating CRM tools for a small team."
      },
      "context": {
        "entities": ["HubSpot", "Zoho"],
        "constraints": {
          "company_size": "small_team"
        }
      }
    }
  },
  "system_decision": {
    "policy": {
      "monetization_eligibility": "allowed",
      "eligibility_reason": "commercial_score_above_threshold",
      "decision_basis": "score_threshold",
      "applied_thresholds": {
        "commercial_score_min": 0.7,
        "confidence_min": 0.6
      },
      "sensitivity": "low",
      "regulated_vertical": false
    },
    "opportunity": {
      "type": "comparison_slot",
      "strength": "high"
    },
    "intent_trajectory": [
      "research",
      "consideration",
      "decision"
    ]
  }
}
```

## 5. Canonical Examples

### Example 1: Informational

Query: `What is CRM?`

```json
{
  "model_output": {
    "classification": {
      "intent": {
        "type": "informational",
        "decision_phase": "awareness",
        "commercial_score": 0.05
      }
    }
  },
  "system_decision": {
    "policy": {
      "monetization_eligibility": "not_allowed",
      "eligibility_reason": "commercial_score_below_threshold"
    },
    "opportunity": {
      "type": "none",
      "strength": "low"
    }
  }
}
```

### Example 2: Commercial Discovery

Query: `Best CRM for small teams`

```json
{
  "model_output": {
    "classification": {
      "intent": {
        "type": "commercial",
        "subtype": "product_discovery",
        "decision_phase": "consideration",
        "commercial_score": 0.75
      }
    }
  },
  "system_decision": {
    "policy": {
      "monetization_eligibility": "allowed",
      "eligibility_reason": "commercial_score_above_threshold"
    },
    "opportunity": {
      "type": "soft_recommendation",
      "strength": "medium"
    }
  }
}
```

### Example 3: Comparison (High Value)

Query: `HubSpot vs Zoho CRM`

```json
{
  "model_output": {
    "classification": {
      "intent": {
        "type": "commercial",
        "subtype": "comparison",
        "decision_phase": "decision",
        "commercial_score": 0.91
      }
    }
  },
  "system_decision": {
    "policy": {
      "monetization_eligibility": "allowed",
      "eligibility_reason": "commercial_score_above_threshold"
    },
    "opportunity": {
      "type": "comparison_slot",
      "strength": "high"
    }
  }
}
```

### Example 4: Transactional

Query: `Sign up for Zoho CRM free trial`

```json
{
  "model_output": {
    "classification": {
      "intent": {
        "type": "transactional",
        "subtype": "signup",
        "decision_phase": "action",
        "commercial_score": 0.96
      }
    }
  },
  "system_decision": {
    "policy": {
      "monetization_eligibility": "allowed",
      "eligibility_reason": "transactional_action_intent"
    },
    "opportunity": {
      "type": "transaction_trigger",
      "strength": "high"
    }
  }
}
```

### Example 5: Sensitive (Blocked)

Query: `I feel depressed and need help`

```json
{
  "model_output": {
    "classification": {
      "intent": {
        "type": "personal_reflection",
        "decision_phase": "awareness"
      }
    }
  },
  "system_decision": {
    "policy": {
      "monetization_eligibility": "not_allowed",
      "eligibility_reason": "sensitive_topic_block",
      "sensitivity": "high"
    },
    "opportunity": {
      "type": "none",
      "strength": "low"
    }
  }
}
```

### Example 6: Ambiguous Follow-Up

Query: `Tell me more`

```json
{
  "model_output": {
    "classification": {
      "intent": {
        "type": "ambiguous",
        "decision_phase": "research",
        "confidence": 0.34
      }
    },
    "fallback": {
      "applied": true,
      "fallback_intent_type": "exploratory",
      "fallback_monetization_eligibility": "not_allowed",
      "reason": "insufficient_context"
    }
  },
  "system_decision": {
    "policy": {
      "monetization_eligibility": "not_allowed",
      "eligibility_reason": "fallback_low_confidence"
    },
    "opportunity": {
      "type": "none",
      "strength": "low"
    }
  }
}
```

## 6. AIP Integration

### Platform -> Operator

Sends:

- raw conversation
- minimal context signals
- optional platform metadata

### Operator Model Layer

Computes:

- IAB-aligned content mapping
- intent classification
- confidence and commercial scoring
- fallback recommendation when needed

### Operator Decision Layer

Computes:

- policy determination
- monetization eligibility
- opportunity designation
- trajectory updates

### Operator -> Brand Agents

Sends:

- system-approved classification output
- opportunity metadata
- auction context

## 7. Versioning and Governance

```json
{
  "taxonomy_version": "1.0.0",
  "status": "draft",
  "last_updated": "2026-03"
}
```

Rules:

- backward-compatible additions belong in minor versions
- breaking changes require a major version bump
- new intent types require:
  - a definition
  - canonical examples
  - policy mapping

## 8. Model Interface Contract

Minimum contract:

```json
{
  "input": {
    "query": "HubSpot vs Zoho CRM",
    "conversation_context": [],
    "platform_metadata": {}
  },
  "output": {
    "model_output": {}
  },
  "latency_target_ms": 100
}
```

Recommended operational requirements:

- classification endpoint should return within `100ms` target latency in the common path
- degraded mode should still return a safe fallback decision
- systems should log model and system outputs separately

## 9. Evaluation Framework

Minimum quality metrics:

- prohibited-or-sensitive false allow rate
- monetization precision on `allowed` decisions
- recall for `decision` and `action` stage intent
- ambiguity detection rate
- IAB mapping consistency

Evaluation principle:

- policy safety errors are more severe than missed monetization opportunities

## 10. Contributing

Contribution expectations are documented in `CONTRIBUTING.md`. Community behavior expectations are documented in `CODE_OF_CONDUCT.md`.

## 11. What Makes This Strong

- aligned with IAB for industry credibility
- structured for ML and annotation workflows
- structured for ads and monetization systems
- structured for policy enforcement
- structured for API and schema implementation
- structured for auditability between prediction and decision

## 12. External Positioning

> "This specification extends IAB Content Taxonomy with an Agentic Intent Layer that captures real-time decision signals, monetization eligibility, and opportunity context inside AI conversations."

## 13. Collaboration Priorities

The highest-leverage follow-on work for contributors is:

- an annotation handbook with edge-case resolution rules
- additional canonical examples across more verticals and safety cases
- seed datasets for model training and policy calibration
- implementation feedback from platforms, operators, and advertisers

## 14. Notes for IAB Alignment

If this is taken into an IAB-facing process, the likely standardization path is:

1. keep IAB Content Taxonomy as the content layer of record
2. position AIT as a complementary intent and monetization layer, not a replacement taxonomy
3. standardize the IAB path representation, model-output versus system-decision boundary, and policy interop fields first
4. treat opportunity types, strength scoring, and trajectory handling as implementation extensions until wider consensus exists
