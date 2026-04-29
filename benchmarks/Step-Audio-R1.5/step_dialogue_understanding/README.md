# step_dialogue_understanding

## Overview

Step-Dialogue-Understanding, abbreviated as Step-DU, is introduced in the Step-Audio-R1.5 paper to evaluate whether a model can answer targeted questions about paralinguistic traits in a conversational setting. While Step-Caption emphasizes comprehensive free-form description, Step-DU focuses on question answering: each sample contains a spoken query from the speaker about their own vocal characteristics, and the model must infer the correct answer solely from the acoustic signal.

The released set contains 87 samples recorded by diverse speakers. The questions cover traits such as age, gender, speaking rate, rhythm, and related vocal attributes. This benchmark is designed to test both perception and reasoning over paralinguistic cues in an interactive dialogue-style setup.

This release package contains:
- 87 public samples in `metadata.jsonl`
- 87 corresponding audio files in `audio/`

## Released files

- `audio/`: audio files referenced by the benchmark metadata
- `metadata.jsonl`: one JSON object per sample
- `prompts/model_user_prompt_template.txt`: the text template used to query the tested model
- `prompts/judge_system_prompt.txt`: the system prompt for the LLM judge
- `prompts/judge_user_prompt_template.txt`: the user prompt template for the LLM judge

## Metadata schema

Each sample in `metadata.jsonl` includes:
- `id`: unique sample id
- `audio_path`: relative path to the released audio file
- `input_question_text`: the text question sent to the tested model
- `recording_requirement_or_qa_category`: reference answer used for evaluation
- `primary_dimension`: top-level task category
- `secondary_dimension`: finer-grained evaluation dimension
- `qa_pair`: original dialogue-style prompt example or collection note
- `question_notes`: optional note field
- `source_audio_name`: original source file name before export

In this released layout, the most important fields for evaluation are:
- `audio_path`
- `input_question_text`
- `recording_requirement_or_qa_category`
- `secondary_dimension`

## Evaluation protocol

Step-DU uses a two-stage evaluation pipeline.

1. Run the tested model on the input audio and question.
   - Load the audio file from the relative path in `audio_path`.
   - Fill `prompts/model_user_prompt_template.txt` with the sample question.
   - In the released prompt file, the template is simply `{question}`.
   - The model should answer the question directly in plain text.

2. Score the model response with an LLM judge.
   - Judge system prompt: `prompts/judge_system_prompt.txt`
   - Judge user prompt template: `prompts/judge_user_prompt_template.txt`
   - Compare the model response against `recording_requirement_or_qa_category`

The judge returns a score in `[0, 1]` together with matched points, missing or incorrect points, and a short reason. The released judge prompt follows these rules:
- prefer `0.0`, `0.5`, and `1.0`
- if the reference answer contains `:` or `：`, only the core label before the colon is evaluated
- if the reference answer contains `/`, any one equivalent option can receive full credit
- if the reference answer contains `；` or `;`, all listed key points must be answered
- semantically equivalent wording is accepted
- contradictory or clearly incorrect content should reduce the score

When constructing the judge input from `prompts/judge_user_prompt_template.txt`, fill the placeholders as follows:
- `{secondary_dimension}`: value from `secondary_dimension`
- `{question}`: value from `input_question_text`
- `{reference_answer}`: value from `recording_requirement_or_qa_category`
- `{model_answer}`: the tested model's response

## Minimal usage workflow

For each sample in `metadata.jsonl`:
1. Read the sample metadata.
2. Load the corresponding audio file.
3. Fill `prompts/model_user_prompt_template.txt` with `input_question_text`.
4. Send the audio plus the filled question prompt to the model under test.
5. Save the model response as plain text.
6. Build the judge request from `prompts/judge_system_prompt.txt` and `prompts/judge_user_prompt_template.txt`.
7. Ask the judge model to return strict JSON output.
8. Average the sample-level scores across all samples to obtain the final Step-DU score.

## Reported model results

The following Step-DU scores are the paper-reported results for mainstream models. Higher is better.

| Model | Step-DU |
| --- | ---: |
| Gemini 3 Flash | 80.46 |
| Gemini 3 Pro | 72.41 |
| qwen3.5-omni-flash | 83.91 |
| qwen3.5-omni-plus | **85.63** |
| Step-Audio-R1 | 64.37 |
| Step-Audio-R1.5 | 82.76 |

Best result is in bold.
