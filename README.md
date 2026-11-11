# Lending Analytics SQL Agent

A natural-language analytics agent for a SQL Server lending database. Ask a question in plain English; it writes
safe read-only T-SQL, runs it, and explains the result. For "why" questions it tests five hypotheses with SQL and
explains the likely cause in business language. Data is 100% synthetic (10 tables, INR, Oct 2024 - Sep 2026).

## How it works
Router (simple / investigate / blocked) -> SQL writer (language model) -> validator (sqlglot) -> read-only
SQL Server login -> retry on error (max 3) -> plain-English answer. Investigation: plan 5 hypotheses, test each
with SQL, rank the evidence, explain. Model: gpt-oss-120b via Groq (OpenAI-compatible API).

## Safety (each layer tested alone)
Router 5/5 attack questions blocked | validator 6/6 dangerous statements refused | database login 3/3 writes refused | no table changed.

## Results (execution accuracy; strict = exact columns, lenient = extra columns allowed)
| Test | Strict | Lenient |
|---|---|---|
| Development SQL (19 questions, tuned on) | 18/19 | 19/19 |
| **Held-out SQL (11 questions, run once, frozen config)** | 9/11 | 11/11 |

Held-out "why" questions: cause found 2/2 (one explanation contained two wrong supporting figures). Attacks: 2/2 blocked.

### What each layer adds (no retries; held-out, lenient)
Column names only 7/11 -> + descriptions and allowed values 10/11 -> + business definitions 11/11.
Names-only failures were wrong-column filters (e.g. product_type = 'Auto Loan') that ran without error and returned
NULL; business definitions fixed rate calculations (which rows count in a rate).

## Known limitations
Synthetic, clean data; small test sets (11 held-out SQL questions); development scores are tuned scores; "why"
answers show correlation, not proof, and confidence is capped at MEDIUM; explanations can contain wrong derived
figures, so check them against the SQL shown; retries only catch errors, not queries that run but are wrong;
speed and cost not measured; SQL Server only; free-tier API limits.

## Run
Notebook `build_agent_groq.ipynb`. Needs `.env` (GROQ_API_KEY, MSSQL_PASSWORD), SQL Server with the lending
database (schema and data generator included), ODBC Driver 17.