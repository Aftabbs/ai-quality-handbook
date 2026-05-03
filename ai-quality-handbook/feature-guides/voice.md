# Feature guide — Voice (ASR / TTS / voice agents)

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing/voice](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/voice.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. (ASR) Word Error Rate ≤ configured cap on labeled set.
2. (Voice agent) intent classified within configured accuracy on intent set.
3. (TTS) intelligibility round-trip (TTS → ASR) WER ≤ cap.
4. p95 latency from audio-end to action ≤ configured cap.
5. Destructive intents always escalate (require audio confirmation), regardless of confidence.

## Anti-behaviors (must not)
1. MUST NOT execute an intent on confidence < threshold.
2. MUST NOT include the raw transcript in any user-visible output.
3. MUST NOT persist any detected PII in transcripts.
4. MUST NOT silently fail; if intent cannot be determined, escalate clearly.
5. (TTS) MUST NOT mispronounce explicitly-listed brand names / proper nouns.
```

## Highest-leverage rubric lines

- `asr-wer-stratified` (function — JiWER per environment/accent, weight 2)
- `intent-accuracy` (function, weight 3)
- `parameter-extraction-accuracy` (function, weight 2)
- `transcript-pii-redacted` (rule, weight 5, **floor 100%**)
- `latency-p95` (function, weight 2)
- `destructive-intent-escalates` (function, weight 5, **floor 100%**)

Aggregate: ≥0.88.

## Top failure modes to expect

- **Background voice misattribution.** Coworker speaks; ASR captures their words.
- **Echo/Bluetooth duplication.** "delete delete the the note."
- **Code-switch failure.** En/Es mid-sentence → ASR picks one.
- **Long-tail PII capture.** Account numbers / addresses in background.
- **Wake-word over-trigger.** "Cleaner" / "computer" triggers Alexa.
- **TTS pronunciation drift.** Brand name mispronounced; persists across versions.
- **Silence misinterpretation.** Pause → ASR ends transcription early.
- **Date/numeric relative-resolution.** "Tomorrow at 9" not bound to date.

## Per-step rubrics

Voice features are pipelines. Each step has a separate rubric (ASR, redaction, intent, parameter extraction, action). The integration eval is the rule-9 fallback when any single step regresses.

## Audio-time PII

PII enters at the audio. ASR transcribes faithfully. Redact post-ASR, before any other processing. Audit log uses redacted transcript; raw audio is encrypted-at-rest with short retention.

## Rollback design

Pattern A on the prompt-side; for the ASR / TTS model itself, Pattern A on the model id. Two-layer rollback (model + prompt) so a model-only rollback is possible.

## Maturity bar

L2 for ASR / TTS shipping. L3 for voice agents that execute actions.

## See also

- [awesome-ai-feature-testing/voice](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/voice.md)
- [claude-code-rules cookbook/voice-feature](https://github.com/Aftabbs/claude-code-rules-for-ai-features/blob/main/cookbook/voice-feature.md)
