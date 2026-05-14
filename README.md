# FitnessBot

FitnessBot is a conversational assistant for helping people get fit and stay fit.

## Messaging strategy

The first supported messenger will be **Discord**, focused on simple one-to-one conversations at first.

The initial experience should feel like a natural chat with a helpful fitness coach, not a command-driven bot. Users should be able to write normal messages such as:

- "I want to get stronger at home";
- "What should I do today?";
- "I finished my workout";
- "My knees hurt after squats";
- "Remind me tomorrow morning".

FitnessBot will use AI for conversational understanding and coaching intelligence. We plan to use **OpenRouter** as the model access layer, with the specific model to be selected later.

The application should still be built around a messenger-neutral core so that future channels, especially WhatsApp, can be added without rewriting the fitness domain logic.


## Discord server onboarding

The launch plan is to run the bot from a VM we control and install it into a Discord server. A user joining that server is enough to make the bot available in the shared server context, assuming the bot has been invited to the server and has permission to read and send messages in the intended channel.

For private coaching, joining the server is only the entry point. The user still needs a conversation path, such as messaging the bot in an allowed server channel, being directed to a bot DM, or otherwise initiating a direct conversation if their Discord privacy settings allow it.

Because the MVP is natural-language driven, the Discord application will need access to message content for the messages it should understand. The implementation should treat this as a Discord adapter concern and keep the fitness logic independent from Discord-specific permissions.

For the initial architectural decision, see [ADR-0001: Discord-first AI chat architecture](docs/architecture/decisions/0001-discord-first-ai-chat-architecture.md).
