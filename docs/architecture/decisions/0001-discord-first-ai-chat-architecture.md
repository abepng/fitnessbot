# ADR-0001: Discord-first AI chat architecture

## Status

Accepted

## Context

FitnessBot needs to be usable from instant messengers. The first production messenger will be Discord.

Telegram is intentionally not part of the initial roadmap. The project may revisit this later, but the current plan is to avoid making Telegram a first-class launch dependency.

WhatsApp remains a likely future channel because of its broad consumer reach, but official programmable WhatsApp integrations require the WhatsApp Business Platform / Cloud API rather than a personal WhatsApp account. That makes WhatsApp a better follow-up integration after the core product experience is validated.

The first version should be simpler than a full community or group-coaching product. We will start with private, natural-language interactions between a user and the bot. Accountability groups, team challenges, and server-channel workflows are explicitly out of scope for the first version.

We also do not want the primary interaction model to be slash commands. Slash commands can be revisited later for shortcuts or admin actions, but the core user experience should be natural conversation backed by AI.

## Decision

Build FitnessBot as a **Discord-first natural chat bot** while keeping the application core independent from Discord-specific concepts.

The bot should use AI for conversational understanding, coaching flow, and response generation. The project will use **OpenRouter** as the model access layer so the exact model can be selected or changed later without rewriting the conversation system.

The codebase should be organized around these boundaries:

1. **Domain core**
   - user profile;
   - fitness goals;
   - constraints, preferences, and available equipment;
   - workout plans;
   - habit tracking;
   - reminders;
   - progress check-ins;
   - safety and coaching rules.

2. **Conversation orchestration**
   - receives natural-language user messages;
   - maintains conversation state;
   - decides when to call AI and when to call deterministic domain services;
   - extracts structured updates from chat, such as completed workouts, goal changes, or pain signals;
   - applies safety rules before sending advice;
   - prepares channel-neutral responses.

3. **AI provider boundary**
   - sends prompts and structured context to OpenRouter;
   - hides provider-specific request and response details from the rest of the application;
   - allows the concrete model to be configured later;
   - supports future model changes, fallbacks, or evaluation runs.

4. **Channel adapters**
   - translate messenger-specific inbound events into a shared internal message format;
   - translate shared outbound responses into messenger-specific API calls;
   - own channel-specific authentication, webhook handling, rate limits, delivery semantics, and interaction primitives.

The first concrete channel adapter should be a Discord adapter. Future WhatsApp support should be implemented as an additional adapter, not by modifying the domain core or conversation logic.

## Initial feature direction

The first version should prioritize a small set of personal-coaching features:

1. **Onboarding through conversation**
   - learn the user's goal;
   - ask about experience level;
   - ask about available equipment and workout location;
   - capture limitations, injuries, pain, or medical constraints;
   - infer a realistic starting plan.

2. **Daily workout guidance**
   - suggest what to do today;
   - adapt the workout to time, equipment, soreness, and previous progress;
   - explain exercises in plain language;
   - offer substitutions when something is unavailable or uncomfortable.

3. **Workout completion tracking**
   - let users report completion naturally;
   - capture skipped workouts without judgment;
   - record sets, reps, duration, perceived effort, or simple completion status;
   - summarize recent consistency.

4. **Habit and reminder support**
   - support reminders for workouts, walking, mobility, hydration, or sleep routines;
   - keep reminder logic in the domain layer so it can later work on WhatsApp;
   - avoid assuming Discord-only interaction patterns.

5. **Progress check-ins**
   - ask periodic questions about energy, soreness, confidence, and goal progress;
   - adjust recommendations based on user feedback;
   - celebrate consistency and small wins.

6. **Safety-aware coaching**
   - avoid diagnosis or medical claims;
   - encourage professional medical advice for pain, injuries, alarming symptoms, or medical uncertainty;
   - bias toward conservative modifications when a user reports discomfort.


## Shared channel contract

Future implementation should introduce a messenger-neutral contract similar to this:

```text
IncomingMessage
- channel
- channelUserId
- conversationId
- messageId
- sentAt
- text
- attachments
- rawEvent

OutgoingMessage
- conversationId
- text
- attachments
- quickReplies
- buttons
- metadata
```

Adapters may support only part of this contract. Unsupported capabilities should degrade gracefully. The core conversation should always be able to fall back to plain text.

## Discord-specific expectations

The Discord adapter should support:

- direct messages for private coaching, reminders, and check-ins;
- normal message events as the main interaction path;
- optional buttons or select menus only when they simplify a natural conversation;
- Discord user identifiers mapped to internal user identities.

Discord-specific objects should not leak into the domain core. They may appear in adapter code and in adapter-owned metadata only.

Slash commands are not part of the first user experience. If added later, they should be shortcuts around the same conversation and domain services rather than a separate product path.

## WhatsApp portability expectations

To preserve a later WhatsApp path, avoid assuming that every channel has Discord-style capabilities.

Design with these constraints in mind:

- WhatsApp conversations are phone-number based, while Discord identities are user-id based.
- WhatsApp has stricter rules for business-initiated conversations and message templates.
- WhatsApp rich interactions differ from Discord components.
- WhatsApp delivery, read receipts, and webhook event shapes are different from Discord events.
- The core bot should be able to produce plain-text prompts and simple button-like choices that can be represented on either platform.

Do not place fitness logic, reminder policy, AI prompting strategy, or user progress tracking inside the Discord adapter. Those behaviors should remain reusable by a future WhatsApp adapter.

## Consequences

### Positive

- Discord can be shipped first without blocking on WhatsApp Business setup.
- The first product is simpler because it focuses on private coaching instead of groups.
- Natural-language interaction keeps the experience closer to a coach than a command-line interface.
- OpenRouter gives room to choose the model later and change models as quality, latency, or cost requirements become clearer.
- The core product remains portable to WhatsApp or other messengers.
- Channel-specific API churn is isolated to adapters.

### Negative

- Natural AI interaction requires stronger safety, evaluation, and prompt/version management than a purely command-based bot.
- A channel abstraction adds some early complexity.
- Some channel behavior will need capability checks instead of one-size-fits-all UI assumptions.
- OpenRouter-specific behavior must be isolated so the app is not tightly coupled to one provider layer.

## Implementation guidance

When the first application code is added, prefer a structure like:

```text
src/
  domain/
  conversation/
  ai/
    openrouter/
  channels/
    shared/
    discord/
    whatsapp/
```

The `whatsapp/` folder can remain absent or contain only planning notes until the WhatsApp integration is actively implemented. The important constraint is that Discord-specific behavior must stay under the Discord adapter boundary.
