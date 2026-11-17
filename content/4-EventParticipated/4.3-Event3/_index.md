---
title: "First Cloud AI Journey – Energetic Meet Up"
date: 2026-10-24
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Summary Report: "BUỔI MEET UP ĐẦY NĂNG LƯỢNG: FIRST CLOUD AI JOURNEY"

| | |
|---|---|
| **Date & Time** | 09:00 – 12:00, October 24, 2026 |
| **Organizer** | AWS Study Group |
| **Location** | 26th Floor, Bitexco Tower, 02 Hai Trieu Street, Saigon Ward, Ho Chi Minh City |
| **Role** | Attendee |

---

### Event Objectives

- Explore the potential of Voice AI and real-time conversational models
- Understand architectures for building Voice AI systems and the latency challenge in voice interaction
- Learn how content is moderated (Guardrails) before an AI replies by voice
- Watch a demo of an AI recruiting assistant that phone-screens candidates automatically

---

### Speakers

- **Kiet Tran** – Reai – Main speaker: Voice AI / Contact Center architecture, Guardrails and the AI recruiting assistant demo
- **AWS Study Group organizers** – Hosted the program, Q&A and the closing mini-game

---

### Key Highlights

#### Latency in voice interaction

- In natural human conversation, the ideal response time is around **200 ms**
- A Voice AI system must respond fast enough that listeners do not feel uncomfortable delays

#### Three Voice AI architecture models

- **Cascade (STT → LLM → TTS):** speech is converted to text, the LLM processes and answers, then the text is converted back to speech
  - Pros: flexible, each module can be replaced independently
  - Cons: latency accumulates across sequential steps and can exceed 500 ms – 1 s
- **Earlier-generation Speech-to-Speech:** converts input audio directly to output audio
  - Pros: very fast
  - Cons: suited to simple tasks, hard to control deep semantic logic
- **New-generation Full Speech-to-Speech:** combines the LLM's understanding with multimodal audio models, reducing latency while keeping natural emotion and intonation

#### Integrating with the Contact Center

- Challenges of connecting AI models to existing call-center infrastructure (SIP trunking, WebRTC, telephony)

#### Guardrails in Voice AI

- LLM output must be moderated **before** it is converted to speech
- Goals: prevent hallucination, sensitive content, deviations from company policy and harmful information
- Set up multiple safety layers on AWS so the assistant stays on the business script

#### Demo of the AI recruiting assistant "Mai"

- Scenario: the assistant "Mai", representing the company "Mê Xanh", phone-screens candidates for an **AWS Cloud Engineer** position
- Flow: automatic dialing → recognize the candidate's speech → LLM evaluates answers against hiring criteria → generate the next question in real time → store results in CRM/Database
- In the demo, the AI asked about AWS experience (EC2, S3, IAM, Lambda) and availability, then ended the call politely
- After the call there is a transcript log, a preliminary skill score and a report for the HR team

#### Technical discussion

- Choosing SDKs/tools for real-time audio streams
- Comparing operating costs across LLMs and speech conversion services
- Managing conversation state in long, complex calls
- Connecting AI agents to enterprise backends via Function Calling / Tool Use

---

### Key Takeaways

**Design Mindset**

- A Voice AI problem starts with **latency** and the **listener's experience**, then moves on to technology choices
- Each architecture involves trade-offs between speed, flexibility and controllability
- Guardrails are a required layer, not an afterthought

**Technical Architecture**

- Understand the STT → LLM → TTS flow and why latency accumulates
- Understand the role of function calling / tool use when an AI agent must work with real systems
- Know what to consider when bringing Voice AI into a call center: telephony, conversation state, cost

**Real-world Application**

- Saw a business process (candidate screening) automated end to end by an AI agent
- Saw how call results are stored and reported so people can make the final decision

---

### Applying to Work

- Apply a **moderate-before-responding** mindset when designing AI applications: consider Guardrails from the start
- When learning more AWS services, estimate the **cost** of AI components (models, speech conversion) before deploying
- Learn more about function calling / tool use to understand how AI agents connect to backend systems

---

### Event Experience

I attended **BUỔI MEET UP ĐẦY NĂNG LƯỢNG: FIRST CLOUD AI JOURNEY** in person from 9:00 AM to 12:00 PM on October 24, 2026 at the 26th Floor, Bitexco Tower, organized by the AWS Study Group community. The main topic was Voice AI and Agentic AI in the Contact Center.

**Learning from the speaker**

Mr. Kiet Tran (Reai) clearly explained Voice AI architectures and why latency decides the user experience. The Guardrails section helped me understand that an AI assistant must have its content controlled before it speaks to users.

**Hands-on demo experience**

I watched the recruiting assistant "Mai" phone-screen a candidate live and saw the results recorded and preliminarily scored. The technical discussion showed me real issues in building Voice AI such as cost, conversation state management and backend integration.

**Networking and discussions**

The Q&A and the closing mini-game reviewed AWS services, language models and conversational architectures.

**Lessons learned**

- AI agents are only useful in practice when they are controlled (Guardrails) and connected to business systems.
- Latency, cost and controllability must be balanced when choosing a Voice AI architecture.

---

### Some Event Photos

![Check-in at the event](/images/4-EventParticipated/4.3-Event3/checkin.jpg)
