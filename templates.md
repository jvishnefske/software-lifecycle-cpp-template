# Artifact Templates

Standard templates for work products produced by each agent.

## Traceability Matrix Template

```yaml
traceability_matrix:
  component: COMP-XXX
  generated: <timestamp>
  
  entries:
    - requirement: REQ-XXX-NNN
      type: functional | safety | performance
      criticality: ASIL-X
      
      architecture:
        component: COMP-XXX
        interface: <interface>
        
      implementation:
        file: src/<file>.cpp
        function: <function>
        lines: <start-end>
      
      verification:
        unit_tests:
          - TC-XXX-NNN
        integration_tests:
          - IT-XXX-NNN
        formal_proofs:
          - FP-XXX-NNN
        review:
          - FR-XXX-NNN
      
      evidence:
        - type: test_result
          reference: <report>
          status: pass | fail
        - type: analysis
          reference: <document>
          status: complete | incomplete
      
      status: fully_traced | partially_traced | not_traced
```

## Deviation Request Template

```yaml
misra_deviation_request:
  id: DEV-XXX-NNN
  date: <date>
  requestor: <agent>
  
  rule:
    number: <MISRA rule>
    category: mandatory | required | advisory
    rule_text: |
      <rule text>
  
  violation:
    file: <file>
    line: <line>
    code: |
      <violating code>
    description: |
      <why this violates the rule>
  
  justification:
    necessity: |
      <why deviation is necessary>
    alternatives_considered:
      - alternative: |
          <what else was considered>
        rejected_because: |
          <why not used>
    
  safety_impact:
    analysis: |
      <how deviation affects safety>
    conclusion: acceptable | unacceptable
  
  compensating_measures:
    - measure: |
        <alternative protection>
      implementation: |
        <how implemented>
      verification: |
        <how verified>
  
  approval:
    status: pending | approved | rejected
    approver: <who>
    date: <when>
    conditions: |
      <any conditions>
```

## Test Report Template

```yaml
test_report:
  component: COMP-XXX
  test_type: unit | integration | system
  execution_date: <timestamp>
  environment:
    platform: <hardware/simulator>
    compiler: <compiler version>
    test_framework: <framework version>
  
  summary:
    total_tests: <n>
    passed: <n>
    failed: <n>
    skipped: <n>
    pass_rate: <percentage>
  
  coverage:
    statement: <percentage>
    branch: <percentage>
    mcdc: <percentage>  # if applicable
  
  results:
    - test_id: TC-XXX-NNN
      name: <test name>
      requirement: REQ-XXX-NNN
      status: pass | fail | skipped
      duration_ms: <n>
      
      # If failed:
      failure:
        assertion: <what failed>
        expected: <expected value>
        actual: <actual value>
        log: |
          <relevant log output>
  
  artifacts:
    log_file: <path>
    coverage_report: <path>
    timing_data: <path>
```

## Safety Case Fragment Template

```yaml
safety_case_fragment:
  component: COMP-XXX
  hazard: HAZ-XXX
  
  goal:
    id: G-XXX
    statement: |
      <safety goal>
  
  context:
    - id: C-XXX
      description: |
        <relevant context>
  
  strategy:
    id: S-XXX
    description: |
      <argument strategy>
  
  claims:
    - id: CL-XXX
      statement: |
        <claim>
      supported_by:
        - type: evidence | sub_claim
          reference: <ID>
  
  evidence:
    - id: E-XXX
      type: test | analysis | review | proof
      description: |
        <what this shows>
      reference: <artifact>
      sufficiency: sufficient | partial | insufficient
  
  assumptions:
    - id: A-XXX
      statement: |
        <assumption>
      justification: |
        <why valid>
  
  status: complete | incomplete
  gaps:
    - <what's missing>
```

## Change Impact Analysis Template

```yaml
change_impact_analysis:
  change_request: CR-XXX
  component: COMP-XXX
  analyst: <agent>
  date: <date>
  
  change_description: |
    <what is being changed>
  
  impact_assessment:
    requirements_affected:
      - REQ-XXX-NNN
    
    architecture_affected:
      - component: COMP-XXX
        impact: |
          <how affected>
    
    code_affected:
      - file: <file>
        impact: |
          <what changes>
    
    tests_affected:
      - test: TC-XXX-NNN
        action: modify | delete | create_new
    
    safety_analysis_affected:
      - fmea_entries:
          - FM-XXX
        fta_affected: yes | no
        safety_case_affected: yes | no
  
  verification_required:
    layers_to_revisit:
      - layer: <n>
        rationale: |
          <why>
  
  risk_of_change:
    level: high | medium | low
    justification: |
      <why this risk level>
```

## Pipeline Report Template

```yaml
pipeline_report:
  project: <project>
  component: COMP-XXX
  pipeline_id: <unique ID>
  
  timeline:
    started: <timestamp>
    completed: <timestamp>
    total_duration: <duration>
  
  layer_summary:
    - layer: 1
      agent: requirements_agent
      status: passed | blocked
      duration: <duration>
      defects_found: <n>
      artifacts_produced:
        - <artifact>
    # ... layers 2-9
  
  defect_summary:
    total_found: <n>
    by_layer:
      layer_1: <n>
      # ...
    by_severity:
      critical: <n>
      major: <n>
      minor: <n>
    
    defects:
      - id: D-XXX
        layer_found: <n>
        severity: critical | major | minor
        description: |
          <description>
        root_cause_layer: <n>
        resolution: |
          <how resolved>
        status: resolved | open
  
  rework_summary:
    total_rework_cycles: <n>
    cycles:
      - from_layer: <n>
        to_layer: <n>
        reason: |
          <why rework needed>
  
  gate_results:
    - gate: 1_2
      result: pass | fail
      attempts: <n>
    # ...
  
  metrics:
    first_pass_yield: <percentage>
    average_defects_per_layer: <n>
    most_effective_layer: <layer>
    
  final_decision: release | hold
  release_conditions:
    - <condition>
  
  lessons_learned:
    - |
      <lesson>
```
