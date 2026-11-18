---
title: "First Cloud AI Journey – Energetic Meet Up"
date: 2026-10-24
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

### First Cloud AI Journey – Energetic Meet Up

| | |
|---|---|
| **Date & Time** | 09:00 – 12:00, October 24, 2026 |
| **Organizer** | AWS Study Group |
| **Location** | 26th Floor, Bitexco Tower, No. 02 Hai Trieu Street, Saigon Ward, Ho Chi Minh City |
| **Role** | Attendee |

---

### 1. Purpose of the Event

The First Cloud AI Journey Meet Up looked at the potential of Voice AI and real-time conversational models, and at how systems that talk with people by voice are built.

The program went through Voice AI architectures, the latency problem in voice conversation, and how content is moderated (Guardrails) before an AI replies. It ended with a demo of an AI recruiting assistant that phone-screens candidates automatically.

---

### 2. Speakers

- **Kiet Tran** – Revve AI – Main speaker: Voice AI / Contact Center architecture, Guardrails and the AI recruiting assistant demo
- **AWS Study Group organizers** – Hosted the program, Q&A and the closing mini-game

---

### 3. Key Highlights

**Latency in Voice Conversation**

- When two people talk, the ideal response time is around **200 ms**.
- A Voice AI system must answer fast enough that the listener does not feel an uncomfortable delay.

**Three Voice AI Architecture Models**

- **Cascade (STT → LLM → TTS):** speech becomes text, the LLM processes it and answers, then the text becomes speech again.
  - Pros: flexible, each module can be replaced on its own.
  - Cons: latency adds up across the steps and can pass 500 ms – 1 s.
- **Earlier-generation Speech-to-Speech:** turns input audio straight into output audio.
  - Pros: very fast.
  - Cons: fits simple tasks only, and deep semantic logic is hard to control.
- **New-generation Full Speech-to-Speech:** combines the understanding of an LLM with multimodal audio models, which lowers latency and keeps natural emotion and intonation.

**Integrating with the Contact Center**

- It is hard to connect AI models to the call-center infrastructure that already exists (SIP trunking, WebRTC, telephony).

**Guardrails in Voice AI**

- The output of the LLM must be moderated **before** it is turned into speech.
- The goal is to stop hallucination, sensitive content, content against company policy and harmful information.
- Build several safety layers on AWS so that the assistant stays on the business script.

**Demo of the AI Recruiting Assistant "Mai"**

- Scenario: the assistant "Mai", for the company "Mê Xanh", phone-screens candidates for an **AWS Cloud Engineer** position.
- Flow: automatic dialing → recognize the candidate's speech → the LLM evaluates answers against the hiring criteria → generate the next question in real time → store results in CRM/Database.
- In the demo, the AI asked about AWS experience (EC2, S3, IAM, Lambda) and availability, then ended the call politely.
- After the call there is a transcript log, a preliminary skill score and a report for the HR team.

**Technical Discussion**

- Choosing SDKs and tools for real-time audio streams.
- Comparing operating cost across LLMs and speech conversion services.
- Managing conversation state in long, complex calls.
- Connecting AI agents to enterprise backends through Function Calling / Tool Use.

---

### 4. What I Learned

**Design Thinking**

- **Start from the listener:** A Voice AI problem starts with **latency** and the **listener's experience**, then moves on to technology choices.
- **Every architecture has trade-offs:** Each model balances speed, flexibility and controllability.
- **Guardrails are required:** They are a layer to have from the start, not something to add later.

**Technical Architecture**

- **Latency adds up:** I understand the STT → LLM → TTS flow and why latency accumulates step by step.
- **Function calling / tool use:** I understand its role when an AI agent has to work with real systems.
- **Bringing it to a call center:** I know that telephony, conversation state and cost need to be considered when Voice AI goes into a call center.

**Real-world Application**

- **End-to-end automation:** I saw a business process (candidate screening) done from start to finish by an AI agent.
- **A human decides at the end:** Call results are stored and reported so that people make the final decision.

---

### 5. Application to Work

- **Moderate before responding:** When I design AI applications, I will think about Guardrails from the start.
- **Estimate cost early:** When I learn more AWS services, I will estimate the **cost** of AI components (models, speech conversion) before deploying.
- **Understand how agents connect to systems:** I will learn more about function calling / tool use to see how AI agents connect to backend systems.

---

### 6. Event Experience

I attended the Meet Up in person from 9:00 AM to 12:00 PM on October 24, 2026 at the 26th Floor, Bitexco Tower, organized by the AWS Study Group community. The main topic was Voice AI and Agentic AI in the Contact Center.

**Learning from the Speaker**

Mr. Kiet Tran (Revve AI) explained Voice AI architectures clearly, and why latency decides the user experience. The Guardrails part helped me understand that an AI assistant needs its content controlled before it speaks to users.

**The Live Demo**

I watched the recruiting assistant "Mai" phone-screen a candidate and saw the results recorded and scored at a first level. The technical discussion showed me real problems in building Voice AI, such as cost, conversation state and backend integration.

**Networking and Exchange**

The Q&A and the closing mini-game helped me review AWS services, language models and conversational architectures.

---

### 7. Key Takeaways

- **Controlled AI is useful AI:** AI agents help in practice only when they are controlled (Guardrails) and connected to business systems.
- **Balance is part of the design:** When choosing a Voice AI architecture, balance latency, cost and controllability.
- **Start from the user:** Technology choices follow from the listener's experience, not the other way around.

---

### 8. Photos

![Check-in at the event](../../images/4-EventParticipated/4.3-Event3/checkin.jpg)

![Session: Hear, Think, Speak – Building a Vietnamese voice-AI contact center on AWS](../../images/4-EventParticipated/4.3-Event3/session.jpg)

![Session slide: Why contact centers, why now](../../images/4-EventParticipated/4.3-Event3/session-contact-center.jpg)

![Group photo of the participants](../../images/4-EventParticipated/4.3-Event3/group.jpg)
