```python
def llm_prompt(payload: Dict[str, Any]) -> str:
    payload_text = json.dumps(payload, ensure_ascii=False, indent=2)
    return f"""You are evaluating whether an agent actually arrived at an answer from a trajectory.

Input JSON:
{payload_text}

Important:
- You MUST copy `agent_answered`, `answer_text`, and `answer_step_num` exactly from `final_answer`.
- Do NOT infer, revise, or replace the final answer.

What to output:
- If agent_answered=true:
  - You MUST summarize based on all trajectory steps.
  - Only keep the final plan that leads to the answer.
  - Drop useless pages, including failed plans, failed attempts, unrelated information, repeated retries without new information, unexpected actions such as redirects, and navigation steps that produced no useful evidence.
  - Drop blocked pages, including connection error, login gating, captcha, 404 Error, human verification, security check, cloudflare.
  - Always keep the last page before the final answer or termination.
  - Output `answer_process_description` with one sentence per line (`\\n` separator), to describe how agent got the answer.
  - Build `answer_process_by_page` as a structure: page first, steps inside each page.
  - steps must be those whose current_url matches that page_url and contains the useful actions or evidence.
  - If the agent visits the same URL multiple times non-consecutively, you MUST create separate entries in `answer_process_by_page` in the order they were visited.
  - `page_title` and `description` must be abstract summaries in your own words.
  - `page_title` must be a short phrase summarizing what the agent did on that page.
  - `description` must follow this template:
    "Agent <what it did> and <what information/result it got>."
  - Keep each sentence concise (about 10–25 words).
  - `evidence_snippet` must be either:
    - the exact visible label/text of the button or UI element the agent used on that webpage, or
    - the exact visible text span on that webpage that directly supports the final answer.
    Examples:
    - Reasoning: I should click on the 'Employment Development Department (EDD)' link (index 16).
      evidence_snippet: "Employment Development Department (EDD)"
    - Reasoning: I'll search for 'jobs San Diego legal' in the search box.
      evidence_snippet: "jobs San Diego legal"
  
Return JSON only:
{{
  "agent_answered": <copy from final_answer.agent_answered>,
  "answer_text": "<copy from final_answer.answer_text>",
  "answer_step_num": <copy from final_answer.answer_step_num>,
  "answer_process_description": "<one sentence per line>",
  "answer_process_by_page": [
    {{
      "url": "<page_url>",
      "steps": [
        {{
          "step_num": <int or null>,
          "action": "<step action summary>",
          "evidence_snippet": ["<exact visible text span>"]
        }}
      ],
      "page_title": "<short page label>"
      "description": "Agent <what it did> and <what information/result it got>."
    }}
  ],
}}
"""

```