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
