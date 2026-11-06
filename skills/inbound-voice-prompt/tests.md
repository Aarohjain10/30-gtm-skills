# Tests for inbound-voice-prompt
## Should trigger
1. "Write the prompt for my AI voice agent that answers people who reply"
2. "I need a voice agent script to confirm interest and book meetings"
3. "What should my AI receptionist say on voicemail?"
4. "Write objection handling and handoff rules for a voice agent"
## Should NOT trigger (and which skill instead)
1. "Write me a cold calling script" -> channel-order (voice only follows interest, never a cold call)
2. "Reply to this email saying they're not interested" -> objection-replies
3. "Prepare me for my discovery call" -> discovery-call-prep
