# Named Entity Recognition in Arabic Mental Health Using LLMs - Reproduction
This repository documents an independent reproduction of a published Arabic mental-health NER baseline using (GPT-4o, LLaMA, ALLaM-7B) from Alfaifi, Alhuzali,
and Alasmari (2026).

Original baseline code: https://github.com/AbeerIbrahimF/MentalQA_NER 

## Reproduction notes

The following operational changes were necessary in the current environment:

1. ALLaM loading was updated for newer `transformers` versions, which reject
   the deprecated `use_auth_token` argument at model construction.
2. LLaMA-4 Scout was accessed through OpenRouter rather than the original Groq
   endpoint, which was unavailable during the rerun.
3. HTTP 429 LLaMA provider failures were retried. Provider-error strings were
   not treated as NER predictions.
4. The `deepseek-chat` alias resolved to DeepSeek-V4.1-Flash during this run,
   rather than the DeepSeek-V3 judge reported in the paper. Judge results are
   therefore reported as a new evaluation, not as an exact replay of the
   published judge verdicts.

## Added diagnostic and recovery code

The following code was added to the working Colab copies during the rerun. It
does not change the released prompts, entity schema, or scoring rules. Its role
is to detect service failures, recover failed API calls, and validate files
before evaluation.

Notebook | Added code | Why it was added |
|:--|:--|:--|
| `RP1_(F)_LLM_as_a_judge.ipynb` | A one-time DeepSeek API check using `requests.post(...)`. | It helps separate a connection problem from an API-key or account-balance problem. A `401` response means the endpoint is reachable, but the request is not authenticated. |
| `RP1_(F)_LLM_as_a_judge.ipynb` | DeepSeek JSON handling: request JSON output, check for an empty reply, and parse the full response with `json.loads(content)`. | This was added after DeepSeek responses caused JSON parsing errors. The original regular expression could select a nested JSON object instead of the complete judge response. |
| `RP1_(F)_LLM_as_a_judge.ipynb` | A repair loop that reruns only judge entries with an `error` verdict and saves the repaired results in `few_shot_ner_judgment_results_repaired.json`. | Some DeepSeek responses were not parsed successfully on the first attempt. |
| `RP_Zero_shot_LLM_as_a_judge.ipynb` | The same DeepSeek JSON handling and targeted repair approach used for the few-shot judge. | It applies the same recovery process to the zero-shot judge results. |
| `RP1_(F)_LLM_as_a_judge.ipynb` | A retry loop for few-shot LLaMA NER outputs that begin with `LLaMA Error:`. It waits longer after each failed attempt. | OpenRouter returned temporary HTTP 429 rate-limit errors. These provider-error messages were recovered instead of being scored as NER predictions. |
| `RP_(F)_ALLaM_NER_(Q).ipynb` | A compatibility update: remove `use_auth_token=True` from the model-loading call and use token-based authentication where needed. | Newer `transformers` versions reject this argument during model loading. |
| `RP1(F)_Few_shot_58_example_evaluation.ipynb` | A helper that merges the 58 gold Drug labels with the three model-output files, repairs one known question-text mismatch, and checks for missing values before evaluation. | It prevents the Drug evaluation from running with an unmatched question or missing prediction.

