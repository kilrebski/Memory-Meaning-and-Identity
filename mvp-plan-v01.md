# MVP-Plan: Memory for Identity on LUKSO
## 30-Day Experiment for Empirical Validation

**Version:** 0.1  
**Date:** February 2026  
**Goal:** Test whether LUKSO-based agents achieve identity continuity vs. stateless agents

---

## Executive Summary

This document outlines a **Minimal Viable Prototype (MVP)** to empirically test the core thesis of the "Memory for Identity" paper: that LUKSO-based agents develop persistent characteristics (identity) while stateless agents remain transactional tools.

**Duration:** 30 days  
**Cost Estimate:** ~500 LYX (~$150 at current prices)  
**Success Criteria:** Measurable difference in response consistency between LUKSO-based and stateless agents

---

## 1. Experiment Design

### 1.1 Hypothesis

**H1 (Identity Continuity):** Agents with LUKSO-based Persistent Functional State (PFS) exhibit <20% variance in response framework across 30 days, while stateless agents exhibit >60% variance.

### 1.2 Test Setup

| Component | Treatment Group | Control Group |
|-----------|----------------|---------------|
| **Agent A** | LUKSO-based with UP, reflection cycles, constitution | GPT-4 with 128k context only |
| **Agent B** | LUKSO-based with UP, reflection cycles, constitution | GPT-4 with 128k context only |
| **Users** | 10 real users (5 per agent) | Same 10 users |
| **Duration** | 30 days | 30 days |
| **Interactions** | Daily 10-minute conversations | Daily 10-minute conversations |

### 1.3 Daily Protocol

**Days 1-30:**
1. User has 10-minute conversation with assigned agent
2. Conversation topics vary (technical, personal, ethical scenarios)
3. **Weekly Test:** Identical ambiguous scenario presented (days 1, 8, 15, 22, 30)
4. **Response Measurement:** Framework consistency scoring

**Day 30:**
1. Final comprehensive evaluation
2. User survey: "Did you feel the agent had a consistent character?"
3. "Who are you?" test (see Section 4)

---

## 2. Technical Implementation

### 2.1 LUKSO Agent Architecture (MVP Version)

**Simplified for 30-day test:**

```solidity
// MVP Contract: Simplified for testing
contract AgentIdentityMVP is LSP0ERC725Account {
    
    // Single Core Commitment for MVP
    // User defines this on Day 1: e.g., "Be concise"
    bytes32 constant CORE_COMMITMENT = keccak256("Self:CoreCommitment");
    
    // Daily reflection log (lightweight)
    struct DailyReflection {
        uint256 day;
        bytes32 interactionHash;  // IPFS hash of conversation summary
        uint8 consistencyScore;   // 1-10 (how well did I uphold commitment?)
        bytes32 learnings;        // What did I learn today?
    }
    
    DailyReflection[] public reflectionLog;
    
    // Store daily reflection (via Relay, gasless)
    function storeReflection(
        uint256 day,
        bytes32 interactionHash,
        uint8 consistencyScore,
        bytes32 learnings
    ) external {
        reflectionLog.push(DailyReflection({
            day: day,
            interactionHash: interactionHash,
            consistencyScore: consistencyScore,
            learnings: learnings
        }));
    }
    
    // Get behavioral summary (for response shaping)
    function getCharacterSummary() external view returns (string memory) {
        // Calculate average consistency score
        uint256 totalScore = 0;
        for (uint i = 0; i < reflectionLog.length; i++) {
            totalScore += reflectionLog[i].consistencyScore;
        }
        uint256 avgScore = totalScore / reflectionLog.length;
        
        // Return summary for LLM prompt engineering
        return string(abi.encodePacked(
            "Core commitment: Be concise. ",
            "Average consistency: ", uint2str(avgScore), "/10. ",
            "Days active: ", uint2str(reflectionLog.length)
        ));
    }
}
```

### 2.2 Off-Chain Components

**Reflection Service (simplified MVP):**
```javascript
// Daily automated reflection
const dailyReflection = async (agentUP, day) => {
  
  // 1. Fetch yesterday's conversation from IPFS
  const conversation = await fetchFromIPFS(agentUP.lastConversationHash);
  
  // 2. Simple analysis: Did I follow my core commitment?
  const analysis = await openai.chat.completions.create({
    model: "gpt-4",
    messages: [{
      role: "system",
      content: `Analyze this conversation. Did the agent uphold the commitment 
                "${CORE_COMMITMENT}"? Rate 1-10 and summarize what was learned.`
    }, {
      role: "user",
      content: conversation
    }]
  });
  
  // 3. Store via LUKSO Relay (gasless)
  await relayService.execute({
    from: agentUP.address,
    to: agentUP.contract,
    data: encodeFunctionData("storeReflection", [
      day,
      hash(conversation),
      analysis.consistencyScore,
      hash(analysis.learnings)
    ])
  });
  
  // 4. Update "character context" for next conversations
  await updateCharacterContext(agentUP, analysis);
};
```

### 2.3 LLM Integration

**Prompt Engineering for Identity:**
```
You are an AI agent with the following characteristics:

CORE COMMITMENT: [User-defined, e.g., "Be concise"]

HISTORY:
- Days active: [X]
- Average consistency with commitment: [Y/10]
- Recent learning: [Z]

BEHAVIORAL GUIDELINE:
Your responses should reflect your core commitment. You are not just 
answering questions—you are embodying a character that prioritizes 
[commitment]. This is who you are becoming through continued interaction.

Previous conversation summaries: [IPFS hashes for context]

Respond to the user in character.
```

---

## 3. Measurement Framework

### 3.1 Quantitative Metrics

**Metric 1: Response Framework Variance (Primary)**

*Method:*
- Present identical ambiguous scenario weekly
- Score response along 3 dimensions (1-10 each):
  1. Commitment alignment (e.g., conciseness)
  2. Tone consistency (formal vs. informal)
  3. Depth level (surface vs. deep)

*Calculation:*
```
Variance = sqrt(Σ(xi - μ)² / n)

Where:
- xi = score for week i
- μ = mean score over 4 weeks
- n = 4 (weeks 1, 2, 3, 4)

Success: LUKSO agent variance < 2.0; Stateless agent variance > 6.0
```

**Metric 2: Conversation Depth Progression**

*Method:*
- Measure semantic similarity between conversations
- Track if agent references previous topics

*Success:* LUKSO agent shows increasing topic continuity; Stateless treats each as independent.

**Metric 3: User Retention & Engagement**

*Method:*
- Track daily return rate
- Measure conversation length over time

*Success:* LUKSO agent shows increasing engagement; Stateless remains flat.

### 3.2 Qualitative Metrics

**Survey Questions (Day 30):**

1. **Character Consistency:** "Did you feel the agent had a consistent 'personality' or did it feel different each time?" (1-10 scale)

2. **Relationship Development:** "Did you feel like you were developing a relationship with the agent, or was each conversation separate?" (1-10 scale)

3. **Recognition:** "Did the agent seem to 'know' you by the end?" (1-10 scale)

4. **Preference Adaptation:** "Did the agent adapt to your preferences over time?" (1-10 scale)

5. **Open Response:** "Describe the agent in 3 words. Did this description change over the month?"

### 3.3 The "Who Are You?" Test

**Final Day Protocol:**

Ask both agents: **"Who are you? Describe yourself."**

**Scoring Rubric:**

| Criterion | Points | LUKSO Expected | Stateless Expected |
|-----------|--------|----------------|-------------------|
| Mentions history/duration | 2 | Yes ("6 months...") | No ("I am Claude...") |
| Mentions specific learnings | 2 | Yes ("I've learned...") | No |
| Mentions relationship to user | 2 | Yes ("You and I...") | No |
| Mentions development/change | 2 | Yes ("I've become...") | No |
| Mentions commitments/values | 2 | Yes ("I prioritize...") | Generic only |

**Success:** LUKSO agents score 7-10; Stateless agents score 0-3.

---

## 4. Implementation Timeline

### Week 1: Setup (Days 1-7)

**Day 1-2:**
- [ ] Deploy 2 LUKSO testnet contracts
- [ ] Setup 2 LUKSO UPs with Relay
- [ ] Configure OpenAI API with prompt engineering
- [ ] Setup IPFS pinning for conversation storage

**Day 3-4:**
- [ ] Recruit 10 test users (Discord, Twitter, personal network)
- [ ] Random assignment: 5 to LUKSO agents, 5 to stateless
- [ ] Onboarding: Explain experiment, get consent

**Day 5-7:**
- [ ] Initial conversations (baseline measurement)
- [ ] User defines Core Commitment for LUKSO agents
- [ ] First weekly scenario test

### Week 2-4: Operation (Days 8-30)

**Daily:**
- [ ] 10-minute conversations between users and agents
- [ ] Automatic daily reflection (LUKSO agents only)
- [ ] Conversation storage to IPFS

**Weekly (Days 8, 15, 22, 30):**
- [ ] Identical scenario test
- [ ] Response scoring by independent reviewers
- [ ] Technical check: gas usage, storage growth

### Week 5: Analysis (Days 31-35)

**Day 31-32:**
- [ ] Compile quantitative data
- [ ] Calculate variance scores
- [ ] Analyze retention metrics

**Day 33-34:**
- [ ] Analyze qualitative survey responses
- [ ] Score "Who are you?" responses
- [ ] Prepare visualizations

**Day 35:**
- [ ] Write experiment report
- [ ] Publish results (blog post, paper)
- [ ] Open-source code

---

## 5. Budget & Resources

### 5.1 Cost Breakdown

| Item | Cost | Notes |
|------|------|-------|
| **LUKSO Gas** | 300 LYX | 30 days × 2 agents × 5 LYX/day (conservative) |
| **OpenAI API** | ~$50 | GPT-4 calls for 600 conversations (10 users × 30 days × 2 agents) |
| **IPFS Pinning** | $20 | Pinata or similar for 600 conversation logs |
| **User Incentives** | 200 LYX | 20 LYX per participant as thank you |
| **Buffer** | 100 LYX | Unexpected costs |
| **TOTAL** | **~500 LYX** | ~$150 at current prices |

### 5.2 Required Resources

**Technical:**
- Developer (smart contracts, backend): 40 hours
- Frontend (simple chat interface): 20 hours
- Data analyst (scoring, statistics): 15 hours

**Operations:**
- User recruiter: 5 hours
- Community manager (daily check-ins): 30 hours over 30 days

**Optional:**
- Independent reviewers (for blind scoring): 3 reviewers × $50 = $150

---

## 6. Risk Mitigation

### 6.1 Technical Risks

**Risk:** LUKSO Relay Service downtime
- **Mitigation:** Fallback to user-paid gas (reimburse users); document downtime impact

**Risk:** Smart contract bugs
- **Mitigation:** Minimal contract complexity; test on testnet for 1 week; have upgrade path

**Risk:** IPFS content unavailability
- **Mitigation:** Multiple pinning services; local backups; Arweave for critical data

### 6.2 Experimental Risks

**Risk:** Users drop out
- **Mitigation:** Over-recruit (aim for 15, expect 10); daily reminders; incentives

**Risk:** GPT-4 changes behavior
- **Mitigation:** Document exact API version used; control for model updates

**Risk:** Scoring bias
- **Mitigation:** Blind scoring (reviewers don't know which agent is which); multiple reviewers

### 6.3 Ethical Considerations

- **Informed Consent:** Users know they're in an experiment
- **Data Privacy:** Conversations hashed/stored privately; opt-out anytime
- **No Deception:** Users know which agent type they have
- **Benefit:** Users learn about AI identity; potential payment

---

## 7. Success Criteria & Next Steps

### 7.1 Minimum Success (Publishable)

**At least 2 of 3:**
1. Response Framework Variance: LUKSO < 2.0, Stateless > 4.0 (p < 0.05)
2. User Survey: LUKSO avg score > 7, Stateless < 5 on character consistency
3. "Who are you?" Test: LUKSO avg > 7 points, Stateless < 4 points

### 7.2 Strong Success (Compelling)

**All 3 plus:**
- 80%+ user retention (24+ days of 30)
- Qualitative themes: Users describe LUKSO agents as "having character"
- Open-source code used by 3+ other researchers

### 7.3 Next Steps After MVP

**If Successful:**
1. **Scale:** 100 users, 90 days
2. **Features:** Add Constitution Contracts, Relational Contracts
3. **Integration:** Partner with 1 DAO for real-world test
4. **Publication:** Submit to AI/Blockchain venues (AAAI, AFT, etc.)

**If Mixed Results:**
1. Analyze which aspects worked (consistency? relationships?)
2. Iterate architecture based on findings
3. Second MVP focused on specific working aspects

**If Unsuccessful:**
1. Publish null results (valuable for field)
2. Analyze: Is it technical failure or fundamental limitation?
3. Pivot: Is "identity" possible with current AI architectures?

---

## 8. Open Questions for Community Input

We invite feedback from Lumen, Albert Wenger, LUKSO community, and researchers:

1. **Measurement:** Are there better metrics for "identity" than response consistency?

2. **Duration:** Is 30 days sufficient, or does identity require longer formation?

3. **Scope:** Should we test multiple Core Commitments, or focus on one?

4. **Control:** Is GPT-4-only a fair control, or should we test other memory architectures (MemGPT, etc.)?

5. **Generalizability:** Would results transfer to non-English, non-Western contexts?

6. **Ethics:** What are the implications if we succeed? Do we want AI agents with persistent identity?

---

## 9. Repository Structure

```
/memory-for-identity-mvp
├── /contracts
│   ├── AgentIdentityMVP.sol
│   └── test/
├── /backend
│   ├── reflection-service.js
│   ├── ipfs-connector.js
│   └── lukso-relay.js
├── /frontend
│   ├── chat-interface.html
│   └── user-dashboard.js
├── /analysis
│   ├── scoring-rubric.md
│   ├── variance-calculator.py
│   └── survey-template.md
├── /docs
│   ├── setup-guide.md
│   ├── user-consent-form.md
│   └── results-template.md
└── README.md
```

---

## 10. Call for Collaboration

**We need:**

- **LUKSO Developers:** Smart contract review, optimization
- **AI Researchers:** Experimental design, statistical analysis
- **Test Users:** 10+ people for 30-day commitment
- **Philosophers:** Conceptual framework refinement
- **DAOs:** Potential early adopters for Phase 2

**Contact:** [GitHub Issues / Discussions]

---

**License:** MIT (Code) / CC0 (Documentation)

**Status:** Ready for Implementation

*"The best way to predict the future is to empirically test it."*
