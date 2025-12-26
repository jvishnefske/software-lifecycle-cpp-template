# MISRA C++ 2023 Control System Agent Framework

A multi-agent system for constructing safety-critical control system software following MISRA C++ 2023 guidelines, incorporating Test-Driven Development (TDD) and the NASA Swiss Cheese Model for defect escape prevention.

## Quick Start

View the agent documentation:

```bash
# View the orchestrator workflow
cat orchestrator.md

# Explore a specific agent (e.g., requirements)
cat 01_requirements_agent.md

# See example usage
cat motor_controller_example.md
```

## Swiss Cheese Model Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          DEFECT INTRODUCTION                                │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 1: REQUIREMENTS AGENT         ○    ○                                 │
│  - Ambiguity detection               │    │                                 │
│  - Completeness checking             ○    │                                 │
│  - Safety requirement derivation          ○                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 2: ARCHITECTURE AGENT              ○         ○                       │
│  - MISRA architectural patterns      ○    │         │                       │
│  - Fault containment regions              │    ○    │                       │
│  - Interface contracts                    ○         │                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 3: TDD TEST AUTHOR AGENT                ○              ○             │
│  - Test-first specification          ○         │              │             │
│  - Boundary condition tests                    │    ○         │             │
│  - Fault injection scenarios                   ○              │             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 4: IMPLEMENTATION AGENT         ○                   ○                │
│  - MISRA C++ 2023 compliant code       │    ○              │                │
│  - Defensive programming               │    │              │                │
│  - Minimal complexity                  ○    │              ○                │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 5: STATIC ANALYSIS AGENT             ○         ○                     │
│  - MISRA rule verification             ○    │         │                     │
│  - Dataflow analysis                        │    ○    │                     │
│  - Complexity metrics                       ○         │                     │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 6: FORMAL VERIFICATION AGENT                   ○              ○      │
│  - Contract verification               ○              │              │      │
│  - Absence of runtime errors                ○         │              │      │
│  - State machine correctness                          ○              │      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 7: INTEGRATION TEST AGENT        ○                        ○          │
│  - Component interaction tests          │    ○                   │          │
│  - Timing verification                  │    │         ○         │          │
│  - Resource exhaustion tests            ○    │                   ○          │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 8: INDEPENDENT REVIEW AGENT              ○              ○            │
│  - Fresh-eyes inspection               ○        │              │            │
│  - Assumption challenging                  ○    │              │            │
│  - Historical defect patterns                   ○              │            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  Layer 9: SAFETY ANALYSIS AGENT                      ○                      │
│  - FMEA/FTA correlation                ○             │         ○            │
│  - Hazard mitigation verification           ○        │         │            │
│  - Safety case evidence                              ○         │            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           RELEASED SOFTWARE                                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

Each layer has independent "holes" (weaknesses), but they are misaligned—defects must pass through all layers to escape.

## Agent Overview

| Agent | Primary Function | Catches Defects Related To |
|-------|------------------|---------------------------|
| Requirements | Validate & formalize requirements | Ambiguity, incompleteness, conflicts |
| Architecture | Design safe structure | Coupling, fault propagation, interfaces |
| TDD Test Author | Write tests before code | Specification gaps, edge cases |
| Implementation | Write MISRA-compliant code | Coding errors, complexity |
| Static Analysis | Automated rule checking | MISRA violations, dataflow issues |
| Formal Verification | Mathematical proofs | Runtime errors, contract violations |
| Integration Test | System-level testing | Interaction bugs, timing issues |
| Independent Review | Human-like inspection | Subtle logic errors, assumptions |
| Safety Analysis | Hazard correlation | Safety requirement gaps |

## Files

- `agents/` - Individual agent system prompts
- `orchestrator.md` - Workflow orchestration prompt
- `artifacts/` - Templates for work products
- `examples/` - Example usage scenarios
