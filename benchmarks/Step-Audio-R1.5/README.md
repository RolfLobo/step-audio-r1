# Step-Audio-R1.5 Open Benchmarks

This directory contains three standalone benchmark releases used in the Step-Audio-R1.5 evaluation. The package under `/data/Step-Audio-R1.5` is an exported release copy; `/data/step-audio-eval` is only a reference codebase for prompt and format alignment and is not part of this release package.

## Included benchmarks

- `step_caption/`: fine-grained audio description benchmark. The model listens to raw audio and produces a natural-language description of speaker characteristics such as gender, age, speaking rate, rhythm, pitch, timbre, emotion, and related paralinguistic traits.
- `step_spqa/`: paralinguistic audio question answering benchmark. It was originally introduced in Step-Audio 2 as an AQAA benchmark and is released here in AQTA form for consistent text-based evaluation across models.
- `step_dialogue_understanding/`: dialogue-style paralinguistic understanding benchmark. The model answers targeted questions about the speaker's own vocal characteristics in an interactive setting.

## Benchmark sizes

- `step_caption`: 905 samples, 905 audio files
- `step_spqa`: 550 samples, 550 audio files
- `step_dialogue_understanding`: 87 samples, 87 audio files

## Package structure

Each benchmark directory contains:
- `audio/`: released audio files
- `metadata.jsonl`: released sample metadata
- `prompts/`: model-side and judge-side prompt files
- `README.md`: benchmark-specific notes

For benchmark-specific evaluation details, see the README inside each benchmark directory.

## Reported model results

The table below lists the paper-reported results on the three released benchmarks. Higher is better.

| Model | Step-Caption | Step-DU | Step-SPQA |
| --- | ---: | ---: | ---: |
| Gemini 3 Flash | 65.12 | 80.46 | 73.80 |
| Gemini 3 Pro | **75.55** | 72.41 | 63.60 |
| qwen3.5-omni-flash | 73.57 | 83.91 | 78.80 |
| qwen3.5-omni-plus | 74.93 | **85.63** | 74.80 |
| Step-Audio-R1 | 70.60 | 64.37 | 74.36 |
| Step-Audio-R1.5 | 71.48 | 82.76 | **79.40** |

Best results are in bold.
