# Example: Motor Controller Component

This example demonstrates using the agent system to develop a simple motor controller.

## Initial Input

```
User Request:
"I need a motor controller that responds to speed commands over CAN bus. 
It should be safe for ASIL-C applications in an agricultural vehicle."
```

## Layer 1: Requirements Agent Output

```yaml
requirement:
  id: REQ-MOT-001
  type: functional
  criticality: ASIL-C
  
  original_text: |
    Motor controller responds to speed commands over CAN bus
  
  formalized:
    shall_statement: |
      The motor controller shall update PWM output to achieve the commanded 
      speed within 10ms of receiving a valid CAN speed command message.
    
    preconditions:
      - Motor controller is in RUNNING state
      - CAN bus operational
    
    postconditions:
      - PWM duty cycle corresponds to commanded speed ±2%
  
  quantification:
    timing:
      response_time_max_ms: 10
      deadline_type: hard
    range:
      min: 0
      max: 6000
      unit: RPM
    tolerance: 2%
  
  derived_safety_requirements:
    - id: REQ-SAF-001
      shall_statement: |
        Upon CAN communication loss exceeding 100ms, the motor controller 
        shall transition to safe state (motor stopped, brake engaged) 
        within 200ms.
    
    - id: REQ-SAF-002  
      shall_statement: |
        The motor controller shall reject commands outside [0, 6000] RPM 
        without changing motor state.

handoff:
  to: architecture_agent
  status: approved_for_architecture
```

## Layer 2: Architecture Agent Output (Excerpt)

```yaml
architecture:
  component:
    id: COMP-MOT-001
    name: MotorController
    criticality: ASIL-C
    
    state_machine:
      initial_state: INIT
      states:
        - name: INIT
          type: normal
        - name: IDLE
          type: normal  
        - name: RUNNING
          type: normal
        - name: SAFE_STOP
          type: safe
        - name: FAULT
          type: error
      
      transitions:
        - from: INIT
          to: IDLE
          trigger: initialization_complete
        - from: IDLE
          to: RUNNING
          trigger: start_command
        - from: RUNNING
          to: IDLE
          trigger: stop_command
        - from: RUNNING
          to: SAFE_STOP
          trigger: can_timeout
        - from: ANY
          to: FAULT
          trigger: fault_detected
    
    interfaces:
      provided:
        - name: IMotorControl
          methods:
            - signature: "Result<void, MotorError> setSpeed(int32_t speed_rpm)"
              timing:
                wcet_us: 50
                deadline_us: 100

handoff:
  to: tdd_test_author_agent
```

## Layer 3: TDD Test Author Output (Excerpt)

```cpp
// test/test_motor_controller.cpp

TEST_F(MotorControllerTest, SetSpeed_ValidCommand_UpdatesPWM) {
    // Arrange
    controller.initialize();
    controller.start();
    
    // Act
    auto result = controller.setSpeed(3000);
    
    // Assert
    EXPECT_TRUE(result.isOk());
    EXPECT_NEAR(controller.currentPWMDuty(), 50, 1); // 50% for 3000 RPM
}

TEST_F(MotorControllerTest, SetSpeed_AboveMax_Rejected) {
    // Arrange  
    controller.initialize();
    controller.start();
    
    // Act
    auto result = controller.setSpeed(7000); // Above 6000 max
    
    // Assert
    EXPECT_TRUE(result.isError());
    EXPECT_EQ(result.error(), MotorError::INVALID_SPEED);
}

TEST_F(MotorControllerTest, CANTimeout_TransitionsToSafeStop) {
    // Arrange
    controller.initialize();
    controller.start();
    controller.setSpeed(3000);
    
    // Act - Simulate CAN timeout
    advanceTime(101); // 101ms > 100ms timeout
    controller.tick();
    
    // Assert
    EXPECT_EQ(controller.state(), State::SAFE_STOP);
    EXPECT_EQ(controller.currentPWMDuty(), 0);
    EXPECT_TRUE(controller.brakeEngaged());
}
```

## Layer 4: Implementation Agent Output (Excerpt)

```cpp
// src/motor_controller.hpp

enum class State : uint8_t {
    INIT = 0U,
    IDLE = 1U,
    RUNNING = 2U,
    SAFE_STOP = 3U,
    FAULT = 4U
};

enum class MotorError : uint8_t {
    NONE = 0U,
    INVALID_SPEED = 1U,
    INVALID_STATE = 2U,
    TIMEOUT = 3U
};

class MotorController final {
public:
    Result<void, MotorError> setSpeed(int32_t speed_rpm) {
        Result<void, MotorError> result;
        
        // Precondition: validate input range
        if ((speed_rpm < 0) || (speed_rpm > MAX_RPM)) {
            result = Result<void, MotorError>::error(MotorError::INVALID_SPEED);
        }
        // Precondition: check state
        else if (m_state != State::RUNNING) {
            result = Result<void, MotorError>::error(MotorError::INVALID_STATE);
        }
        else {
            m_targetSpeed = speed_rpm;
            updatePWM();
            result = Result<void, MotorError>::ok({});
        }
        
        return result;
    }
    
    void tick() {
        // Check CAN timeout
        if (m_state == State::RUNNING) {
            uint32_t const elapsed = getTimeSinceLastCAN();
            if (elapsed > CAN_TIMEOUT_MS) {
                transitionToSafeStop();
            }
        }
    }

private:
    static constexpr int32_t MAX_RPM{6000};
    static constexpr uint32_t CAN_TIMEOUT_MS{100U};
    
    State m_state{State::INIT};
    int32_t m_targetSpeed{0};
    
    void transitionToSafeStop() {
        m_targetSpeed = 0;
        updatePWM();
        engageBrake();
        m_state = State::SAFE_STOP;
    }
};
```

## Layer 5: Static Analysis Output (Excerpt)

```yaml
misra_compliance:
  summary:
    mandatory_rules:
      total: 15
      compliant: 15
      violations: 0
    required_rules:
      total: 42
      compliant: 42
      violations: 0

dataflow_analysis:
  issues: []  # No issues found

complexity_metrics:
  functions:
    - name: setSpeed
      cyclomatic_complexity: 3
      status: pass
    - name: tick
      cyclomatic_complexity: 2
      status: pass

overall_assessment:
  status: pass
```

## Layer 6: Formal Verification Output (Excerpt)

```yaml
contract_verification:
  functions:
    - name: setSpeed
      preconditions:
        - condition: "speed_rpm in [0, MAX_RPM]"
          status: proven
          proof: "Explicit check at line 12 guards all paths"
      postconditions:
        - condition: "m_targetSpeed == speed_rpm OR error returned"
          status: proven

runtime_error_freedom:
  verifications:
    - category: integer_overflow
      operations:
        - location: motor_controller.cpp:15
          expression: "m_targetSpeed = speed_rpm"
          status: safe
          proof: "Input validated to [0, 6000], fits int32_t"

state_machine_verification:
  forbidden_transitions:
    - from: SAFE_STOP
      to: RUNNING
      status: proven_unreachable
      proof: "No transition defined from SAFE_STOP to RUNNING"
```

## Layer 7: Integration Test Output (Excerpt)

```yaml
timing_verification:
  task_timing:
    - task: MotorControlLoop
      wcet_budget_us: 100
      wcet_measured_us: 47
      wcet_status: pass
      deadline_met: true

fault_injection_tests:
  tests:
    - id: FI-MOT-001
      name: CAN_Timeout_SafeStop
      fault_injected:
        description: "Block all CAN messages"
      expected_response:
        detection_time_ms: 100
        safe_state: SAFE_STOP
      actual_response:
        detection_time_ms: 101
        end_state: SAFE_STOP
      result: pass
```

## Layer 8: Independent Review Output (Excerpt)

```yaml
adversarial_analysis:
  assumption_challenges:
    - assumption: "CAN bus provides message timing"
      challenge: "What if CAN controller hangs?"
      validation_status: validated
      how_addressed: "Watchdog timer independent of CAN"

findings_summary:
  critical:
    count: 0
  major:
    count: 1
    items:
      - id: FR-MOT-001
        title: "Missing brake status feedback"
        description: |
          Brake engagement is commanded but success is not verified
        recommendation: |
          Add brake status readback with timeout
        blocking: false

overall_assessment:
  review_verdict: conditional_approve
  conditions_for_approval:
    - condition: "Address brake feedback finding"
```

## Layer 9: Safety Analysis Output (Excerpt)

```yaml
hazard_traceability:
  hazards:
    - id: HAZ-MOT-001
      description: "Uncontrolled motor operation"
      safety_requirements:
        - req_id: REQ-SAF-001
          implementation_correct: yes
          test_coverage:
            - FI-MOT-001
          coverage_adequate: yes
      mitigated_risk: acceptable

release_recommendation:
  recommendation: conditional_release
  conditions:
    - condition: "Brake feedback finding addressed"
      status: not_met
  outstanding_risks:
    - risk: "Brake engagement not verified"
      mitigation_in_operation: "Pre-flight brake test procedure"
```

## Final Pipeline Result

```yaml
pipeline_report:
  component: COMP-MOT-001
  final_decision: conditional_release
  
  layer_summary:
    - layer: 1, status: passed, defects_found: 0
    - layer: 2, status: passed, defects_found: 0
    - layer: 3, status: passed, defects_found: 0
    - layer: 4, status: passed, defects_found: 0
    - layer: 5, status: passed, defects_found: 0
    - layer: 6, status: passed, defects_found: 0
    - layer: 7, status: passed, defects_found: 0
    - layer: 8, status: passed, defects_found: 1 (major)
    - layer: 9, status: passed, defects_found: 0
  
  swiss_cheese_effectiveness:
    # One defect (brake feedback) caught at Layer 8
    # Would have escaped if only earlier layers used
    # Demonstrates value of independent review layer
```

## Usage Notes

This example shows how a simple motor controller flows through all 9 layers. Key observations:

1. **Requirements Agent** formalized vague requirements and derived safety requirements
2. **Architecture Agent** defined clean state machine and interfaces
3. **TDD Test Author** wrote tests before implementation existed
4. **Implementation Agent** wrote MISRA-compliant code that passed tests
5. **Static Analysis Agent** verified MISRA compliance automatically
6. **Formal Verification Agent** proved absence of runtime errors
7. **Integration Test Agent** verified real timing and fault response
8. **Independent Review Agent** caught a design gap (brake feedback)
9. **Safety Analysis Agent** correlated with hazard analysis

The brake feedback issue found at Layer 8 demonstrates the Swiss Cheese model working—earlier layers had a shared blind spot that the independent review caught.
