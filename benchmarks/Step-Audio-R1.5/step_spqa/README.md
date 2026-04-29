# step_spqa

## Overview

StepEval-Audio-Paralinguistic, abbreviated as Step-SPQA, was originally introduced in Step-Audio 2 as an AQAA benchmark, where both the query and the answer are spoken audio. For Step-Audio-R1.5, it is converted to the AQTA format, meaning the query remains grounded in audio understanding while the model output is evaluated as text. This conversion keeps the original paralinguistic understanding tasks unchanged while enabling consistent text-based evaluation across different models.

In the released layout here, each sample uses an `*_input.wav` file that already contains the spoken query together with the source audio context. As a result, the official released evaluation mode is audio-only input at inference time, with no extra text question required by default.

This release package contains:
- 550 public samples in `metadata.jsonl`
- 550 corresponding audio files in `audio/`
- 11 paralinguistic sub-tasks with 50 samples each

The released sub-tasks are:
- `age`
- `emotions`
- `event`
- `gender`
- `pitch`
- `rhythm`
- `scene`
- `speed`
- `vocalsound`
- `voice_styles`
- `voice_tone`

## Released files

- `audio/`: released audio files organized by task type
- `metadata.jsonl`: one JSON object per sample
- `prompts/model_user_prompt_template.txt`: the model-side text template
- `prompts/model_usage_note.txt`: note explaining the released audio-only inference mode
- `prompts/judge_prompts/`: task-specific semantic-match judge prompts
- `prompts/judge_prompts/prompt_map.json`: mapping from task type to prompt file

## Metadata schema

Each sample in `metadata.jsonl` includes:
- `id`: unique sample id used for evaluation
- `audio_id`: original audio identifier
- `audio_path`: relative path to the released audio file
- `task_type`: one of the 11 released sub-task names
- `task_name`: human-readable task name
- `label`: short canonical label for the answer
- `reference_answer`: natural-language reference answer used in evaluation
- `question`: runtime text question; empty in the released audio-only setup
- `audio_only_input`: whether the released audio already contains the spoken query
- `legacy_question`: original spoken question text from the earlier format
- `source_text`: transcript of the source speech content

In the released benchmark, the most important fields for inference and evaluation are:
- `audio_path`
- `task_type`
- `reference_answer`
- `label`
- `audio_only_input`
- `legacy_question`

## Evaluation protocol

Step-SPQA uses text-answer evaluation over audio-grounded paralinguistic tasks.

1. Run the tested model on the input audio.
   - Load the audio file from the relative path in `audio_path`.
   - If `audio_only_input=true`, use the released audio as the full query input.
   - In this released package, the default text prompt is empty because the spoken question is already embedded in the audio.
   - `prompts/model_user_prompt_template.txt` is kept for format completeness, but in the official released setup it resolves to an empty question.
   - If you want a text-assisted ablation, you may use `legacy_question` as the user prompt, but that is not the default released evaluation mode.

2. Score the model response.
   - First perform normalized exact match against `reference_answer` and `label`.
   - If exact match fails, use an LLM judge with the task-specific prompt selected from `prompts/judge_prompts/`.
   - The mapping from `task_type` to prompt file is defined in `prompts/judge_prompts/prompt_map.json`.

The released judge prompts ask for a binary semantic decision, returning only `YES` or `NO`. Different task types use different matching criteria. For example:
- `age`: allows approximate age-range agreement
- `gender`: requires consistent gender identification
- `speed`: allows neighboring speaking-rate categories
- `emotions`: uses coarse semantic grouping of emotion labels
- `vocalsound` and `event`: allow semantically related sound-event matches

A sample receives score `1.0` if it passes exact match or the LLM judge returns `YES`; otherwise it receives `0.0`. Final benchmark performance is the average accuracy over all 550 samples.

## Minimal usage workflow

For each sample in `metadata.jsonl`:
1. Read the sample metadata.
2. Load the corresponding audio file from `audio_path`.
3. Send the audio to the model under test.
4. By default, do not add extra text if `audio_only_input=true`.
5. Save the model response as plain text.
6. Try normalized exact match against `reference_answer` and `label`.
7. If exact match fails, pick the task-specific judge prompt according to `task_type` and `prompts/judge_prompts/prompt_map.json`.
8. Fill the selected judge prompt with the reference answer and the model answer.
9. Ask the judge model to return only `YES` or `NO`.
10. Average sample-level correctness over all samples to obtain the final Step-SPQA score.

## Reported model results

The following Step-SPQA scores are the paper-reported results for mainstream models. Higher is better.

| Model | Step-SPQA |
| --- | ---: |
| Gemini 3 Flash | 73.80 |
| Gemini 3 Pro | 63.60 |
| qwen3.5-omni-flash | <u>78.80</u> |
| qwen3.5-omni-plus | 74.80 |
| Step-Audio-R1 | 74.36 |
| Step-Audio-R1.5 | **79.40** |

Best result is in bold; second-best is underlined.
