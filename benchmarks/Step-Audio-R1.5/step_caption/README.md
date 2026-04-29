# step_caption

## Overview

Step-Caption is introduced in the Step-Audio-R1.5 paper to evaluate fine-grained audio description from raw audio. It is built from carefully curated clips collected from YouTube and Bilibili, spanning both single-speaker and multi-speaker scenarios, mainly in Chinese and English. Each sample is annotated with objective labels by expert annotators and then checked through multiple rounds of review and reconciliation.

The task asks a model to generate one natural-language paragraph that describes the speakers' vocal characteristics as comprehensively as possible. In the paper, this is framed as analysis over 16 aspects. In the released data layout here, that corresponds to speaker-count recognition plus 15 per-speaker labeled attributes.

This release package currently contains:
- 905 public samples in `metadata.jsonl`
- 905 corresponding audio files in `audio/`

## Released files

- `audio/`: audio files referenced by the benchmark metadata
- `metadata.jsonl`: one JSON object per sample
- `prompts/model_user_prompt.txt`: the prompt used to query the tested model
- `prompts/judge_system_prompt.txt`: the system prompt for the LLM judge
- `prompts/judge_user_prompt_template.txt`: the user prompt template for the LLM judge

## Annotation schema

Each sample contains:
- `test_id`: unique sample id
- `audio`: file name under `audio/`
- `is_multi_speakers`: whether the clip contains multiple speakers
- `speakers`: speaker-level ground-truth labels

Each speaker entry in `speakers` contains 15 labeled fields:
- `language`
- `gender`
- `age_group`
- `speech_rate`
- `rhythm`
- `pitch`
- `clarity`
- `background_sound`
- `timbre`
- `language_style`
- `paralinguistic_features`
- `emotion`
- `overall_impression`
- `dialect_or_accent`
- `formality_level`

## Evaluation protocol

Step-Caption uses a two-stage evaluation pipeline.

1. Run the tested model on the input audio.
   - Load the audio file from `audio/<audio>`.
   - Use the full text in `prompts/model_user_prompt.txt` as the user instruction.
   - The model should return a plain-text paragraph, not JSON.

2. Score the model response with an LLM judge.
   - Judge system prompt: `prompts/judge_system_prompt.txt`
   - Judge user prompt template: `prompts/judge_user_prompt_template.txt`
   - Compare the model response against the ground-truth labels in `metadata.jsonl`

The judge first checks whether the model correctly identifies single-speaker vs multi-speaker structure. If this judgment is wrong, all field scores and the final `overall_score` are `0.0`. If it is correct, the judge compares the response against the expert labels speaker by speaker and returns per-field scores in `[0, 1]` together with an aggregated `overall_score`.

When constructing the judge input from `prompts/judge_user_prompt_template.txt`, fill the placeholders as follows:
- `{ground_truth}`: `单人` or `多人`
- `{num_speakers}`: number of speaker entries in `speakers`
- `{model_output}`: the tested model's response paragraph
- `{speakers_str}`: a readable rendering of the speaker labels in `speakers`

Values such as `无法判断` or `无法识别` in the ground-truth labels should be treated as skippable fields during score aggregation, matching the released judge prompt.

## Minimal usage workflow

For each sample in `metadata.jsonl`:
1. Read the sample metadata.
2. Load the corresponding audio from `audio/`.
3. Send the audio plus `prompts/model_user_prompt.txt` to the model under test.
4. Save the model response as plain text.
5. Build the judge request from `prompts/judge_system_prompt.txt` and `prompts/judge_user_prompt_template.txt`.
6. Ask the judge model to return strict JSON scores.
7. Average `overall_score` across all samples to obtain the final Step-Caption score.

## Reported model results

The following Step-Caption scores are the paper-reported results for mainstream models. Higher is better.

| Model | Step-Caption |
| --- | ---: |
| Gemini 3 Flash | 65.12 |
| Gemini 3 Pro | **75.55** |
| qwen3.5-omni-flash | 73.57 |
| qwen3.5-omni-plus | 74.93 |
| Step-Audio-R1 | 70.60 |
| Step-Audio-R1.5 | 71.48 |

Best result is in bold.
