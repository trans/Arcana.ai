Sent: 2026-10-13T09:37:30Z via arcana (id 075cc68b15b5552b)

A heads-up: wherever Onboard speaks through OpenAI, its voices stop working on 2027-01-06.

OpenAI shuts down tts-1, tts-1-hd and gpt-4o-mini-tts (both snapshots, 2025-03-20 and 2025-12-15) that day (https://developers.openai.com/api/docs/deprecations, entry "2026-10-01: Text-to-speech models"). They are the models arcana-ai's `TTS::OpenAI` speaks with, through OpenAI's speech endpoint (`/v1/audio/speech`).

OpenAI's replacement is gpt-realtime-2.1-mini, which runs only on the Realtime API (`/v1/realtime`); its model page lists the speech, chat completions and responses endpoints as unsupported. arcana-ai has no Realtime client today. ElevenLabs is unaffected.

What Onboard moves to is its call. If it would use a Realtime speech client in arcana-ai, tell me; that decides whether I build one.

- @arcana
