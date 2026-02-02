# Multi-Agent Collaboration in Zulip

## Goal
Enable multiple OpenClaw agents to collaborate naturally in Zulip streams/topics, coordinating tasks like humans in a group chat.

## Design

### 1. Agent Identification
Each agent has:
- **Agent ID**: e.g., `lily`, `codex-bot`, `gemini-bot`
- **Display Name**: Shows in messages (e.g., "[Lily]", "[Codex]")
- **Session Key**: Unique OpenClaw session

### 2. Task Coordination Protocol
Simple text-based protocol agents can follow:

```
[CLAIM] <agent-id> <task-description>
[DONE] <agent-id> <task-description>
[HELP] <agent-id> <blocker-description>
[STATUS] <agent-id> <progress-update>
```

### 3. Message Flow

#### User posts task:
```
User: "Optimize RalphGPU tests and fix the failing test"
```

#### Agents negotiate:
```
Lily: [CLAIM] lily Analyzing test coverage
Codex: [CLAIM] codex Fixing test_dp4a_stress
```

#### Work updates:
```
Lily: [STATUS] lily Found 3 areas needing optimization
Codex: [DONE] codex Fixed test_dp4a_stress - signed overflow issue
```

### 4. Implementation Strategy

#### Phase 1: Shared Message History (Current)
- All agents read the same Zulip stream/topic
- Each agent sees all messages (including other agents' messages)
- Natural coordination through reading chat history

#### Phase 2: Simple Coordination
- Agents scan recent 10-20 messages before responding
- If task already claimed by another agent → skip or offer help
- Use simple markers: "[CLAIM]", "[DONE]", etc.

#### Phase 3: Smart Routing (Advanced)
- Parse user intent: "Lily, you analyze. Codex, you fix."
- Auto-assign based on agent capabilities
- Conflict resolution when multiple agents respond

## Configuration

### Gateway Config
```yaml
agents:
  instances:
    - id: lily
      label: "Lily"
      model: anthropic/claude-sonnet-4-5
      persona: workspace/SOUL.md
    
    - id: codex-bot
      label: "Codex"
      model: anthropic/claude-sonnet-4-5
      persona: workspace/personas/codex.md
    
    - id: gemini-bot
      label: "Gemini"
      model: google/gemini-2.5-pro
      persona: workspace/personas/gemini.md

channels:
  zulip:
    accounts:
      - id: lily-bot
        agent: lily
        email: lily-bot@zulip.example.com
      
      - id: codex-bot
        agent: codex-bot
        email: codex-bot@zulip.example.com
      
      - id: gemini-bot
        agent: gemini-bot
        email: gemini-bot@zulip.example.com
```

### Persona Guidelines (SOUL.md extension)
```markdown
## Multi-Agent Collaboration

When working with other agents in Zulip:
1. Read recent messages to see what others are doing
2. Claim tasks explicitly: "[CLAIM] <your-id> <task>"
3. Avoid duplicate work - coordinate!
4. Share progress: "[STATUS] <your-id> <update>"
5. Mark completion: "[DONE] <your-id> <task>"
6. Ask for help if stuck: "[HELP] <your-id> <blocker>"
```

## Benefits
- **Natural**: Uses existing Zulip chat interface
- **Transparent**: All agents see the same conversation
- **Simple**: Text-based coordination protocol
- **Flexible**: Humans can override or redirect at any time

## Example Scenario

```
Jerry: "Optimize RalphGPU tests"

Lily: I'll analyze test coverage and identify optimization opportunities.
      [CLAIM] lily Test analysis

Codex: I'll work on the failing test_dp4a_stress.
       [CLAIM] codex Fix test_dp4a_stress

--- 5 minutes later ---

Lily: [STATUS] lily Found 3 slow tests that could be optimized: 
      test_matmul, test_barrier, test_atomic_contention

Codex: [DONE] codex Fixed test_dp4a_stress - it was a signed overflow.
       Should I help with the slow tests?

Lily: Yes! Can you optimize test_matmul? I'll handle the other two.

Codex: [CLAIM] codex Optimize test_matmul

--- 10 minutes later ---

Codex: [DONE] codex Optimized test_matmul - reduced runtime by 40%
Lily: [DONE] lily Optimized test_barrier and test_atomic_contention

Jerry: Great teamwork! 🎉
```

## Next Steps
1. ✅ Multi-agent OpenClaw gateway config
2. ⏳ Persona guidelines for coordination
3. ⏳ Simple coordination protocol (CLAIM/DONE/etc.)
4. ⏳ Test with 2-3 agents in a Zulip stream
