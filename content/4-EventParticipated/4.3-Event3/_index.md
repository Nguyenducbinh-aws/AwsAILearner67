---
title: "First Cloud AI Journey – Energetic Meet Up"
date: 2026-10-24
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

### Summary Report

| | |
|---|---|
| **Date & Time** | 09:00 – 12:00, October 24, 2026 |
| **Organizer** | Amazon Web Services Viet Nam Company Limited |
| **Location** | 26th Floor, Bitexco Tower, No. 02 Hai Trieu Street, Saigon Ward, Ho Chi Minh City |
| **Role** | Attendee |

---

### 1. Purpose of the Event

The First Cloud AI Journey Meet Up looked at the potential of Voice AI and real-time conversational models, and at how systems that talk with people by voice are built.

The Meet Up ran for three hours with five sessions: a Voice AI contact center, resource sharing with AWS Resource Access Manager, security testing with AWS Security Agent, Amazon Bedrock AgentCore, and Kiro Crew.

The program went through Voice AI architectures, the latency problem in voice conversation, and how content is moderated (Guardrails) before an AI replies. The first session ended with a demo of an AI recruiting assistant that phone-screens candidates automatically. The later sessions moved on to multi-account management, infrastructure security assessment and running AI agents in production.

---

### 2. Speakers

- **Kiet Tran** – Revve AI – Session 1: Voice AI / Contact Center architecture, Guardrails and the AI recruiting assistant demo
- **Long Huynh** – Styl Solution – Session 2: AWS Resource Access Manager
- **Thinh Nguyen** – Styl Solution – Session 3: AWS Pentest with AWS Continuum / Security Agent
- **Xuan An** – Leader of AWS Student Builder Group HCMUTE – Session 4: Amazon Bedrock AgentCore
- **Khoa Kieu** (Tech Lead, ACB) and **Nghi Danh** (AWS Community Builder) – Session 5: Kiro Crew

---

### 3. Key Highlights

#### Session 1: Hear, Think, Speak – Building a Vietnamese voice-AI contact center on AWS

**Latency in Voice Conversation**

- When two people talk, the ideal response time is around **200 ms**.
- A Voice AI system must answer fast enough that the listener does not feel an uncomfortable delay.
- The architecture presented brings latency down to about **1.8 seconds** so the conversation stays smooth.

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

#### Session 2: AWS Resource Access Manager

- Share resources safely between accounts in the same AWS Organization, without making copies in each account.
- This saves cost and brings network management into one place.
- Combine IAM policies with SCPs so that no account or user goes beyond the permission ceiling that was set.

#### Session 3: AWS Pentest with AWS Continuum / Security Agent

- Assess infrastructure security early, inside the development process.
- Review Terraform code and attach the review to Pull Requests on GitLab/GitHub to catch errors before merging.
- The Security Agent plays a real user and runs automated pentests on SIT/UAT environments before the real deployment.

#### Session 4: Amazon Bedrock AgentCore

- Two platforms for taking agents to production: **Agent Runtime** and the **Memory** system.
- Agent Runtime runs on MicroVMs on EC2 Spot, which keeps cost low.
- Memory has two tiers: Short-term (synchronous) and Long-term (asynchronous), so the agent understands the user's context in depth.

#### Session 5: Kiro Crew

The problem was running an AI agent 24/7 to help with code review and system checks. The technical Q&A at the end had three notable topics:

- **Artifacts:** Kiro Crew collects real job-posting data and builds statistical charts right inside the chat window.
- **Context window management:** an Auto Compact mechanism runs when the context reaches 70–90%, and a Break down context feature shows the token use of each component.
- **Memory and Guardrails:** clean up old memory so cost does not grow, and add a security layer so the AI does not delete infrastructure or leak server secrets and credentials on its own.

---

### 4. What I Learned

Session 1 gave me a way to approach a Voice AI problem: start from latency and the listener's experience, then choose the technology. The three architectures trade speed, flexibility and controllability differently, and in the cascade model latency adds up at each STT, LLM and TTS step. Guardrails belong in the design from the start.

On governance, I learned that AWS RAM lets accounts use the same resource instead of copying it, and that an SCP sets a permission ceiling for the whole organization above IAM policies. The Terraform and Pull Request review in session 3 showed that infrastructure errors can be caught before deployment instead of after.

AgentCore separates running the agent (Runtime) from remembering context (short-term and long-term Memory). Context windows and old memory both raise token use, so Auto Compact and regular cleanup are needed. Besides content moderation, an agent also needs a block that stops it from deleting infrastructure or leaking credentials.

I also saw function calling / tool use connect an AI agent to real systems. In the demo, the recruiting assistant ran candidate screening from start to finish, and the results were stored and reported so that people make the final decision. Bringing Voice AI into a call center means taking telephony, conversation state and cost into account.

---

### 5. Application to Work

- I will think about Guardrails from the start when I design AI applications.
- When I learn more AWS services, I will estimate the cost of AI components (models, speech conversion) before deploying.
- I will learn more about function calling / tool use to see how AI agents connect to backend systems.
- In Cost Guard I will keep the rule that each Lambda function has one IAM role that can only stop the target instance, in line with the permission ceiling from session 2 and the block that stops agents from deleting infrastructure in session 5.

---

### 6. Event Experience

I attended the Meet Up in person from 9:00 AM to 12:00 PM on October 24, 2026 at the 26th Floor, Bitexco Tower, organized by Amazon Web Services Viet Nam Company Limited. The day had five sessions, opening with Voice AI and Agentic AI in the Contact Center.

Mr. Kiet Tran (Revve AI) explained the Voice AI architectures and why latency decides the user experience. His Guardrails part showed me that an AI assistant's output has to be controlled before it speaks to users. The two speakers from Styl Solution covered resource sharing and security testing, and Mr. Xuan An introduced AgentCore. The last session, on Kiro Crew, interested me most, because it dealt with keeping an AI agent running continuously while still controlling cost and permissions.

I watched the recruiting assistant "Mai" phone-screen a candidate and saw the results recorded and scored at a first level. The technical discussion covered real problems in building Voice AI: cost, conversation state and backend integration. The Q&A and the closing mini-game helped me review AWS services, language models and conversational architectures.

---

### 7. Key Takeaways

AI agents help in practice only when they are controlled with Guardrails and connected to business systems. Choosing a Voice AI architecture means balancing latency, cost and controllability, starting from the listener's experience. The common idea from SCPs in session 2 to Guardrails and memory management in session 5 is to set a ceiling on permissions and cost before the system runs.

---

### 8. Photos

![Check-in at the event](../../images/4-EventParticipated/4.3-Event3/checkin.jpg)

![Session: Hear, Think, Speak – Building a Vietnamese voice-AI contact center on AWS](../../images/4-EventParticipated/4.3-Event3/session.jpg)

![Session slide: Why contact centers, why now](../../images/4-EventParticipated/4.3-Event3/session-contact-center.jpg)

![Group photo of the participants](../../images/4-EventParticipated/4.3-Event3/group.jpg)
