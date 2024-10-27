# llama-3.2-benchmark
A test on LLaMA 3.2 on MMLU benchmark.

## What is MMLU?

MMLU (which stands for Massive Multitask Language Understanding) is a benchmark task that tests model understanding in question-answering format. It contains 14,042 questions on 57 different fields across STEM, humanities, and social sciences.


## My replication results

| Run | Accuracy (Micro/Macro)       | Remarks                                                                                                   |
|-----|------------------------------|-----------------------------------------------------------------------------------------------------------|
| 1   | 21.3% (micro)                | Simple 2-shot, "Answer: [Answer]" prompting                                                               |
| 2   | 15.7% (micro)                | 5-shot, prompts coming from dev set                                                                       |
| 3   | 30.5% (micro)                | Previous setting + temperature = 0.1                                                                      |
| 4   | 35.7% (micro)                | Improved JSON formatting, increased output max tokens                                                     |
| 5   | 36.3% (micro)                | Cleaned prev prompt, but added final instruction prompt                                                   |
| 6   | 37.1% (micro), 39.0% (macro) | "Compartmentalized" shots in prompt                                                                       |
| 7   | 42.9% (micro), 43.7% (macro) | "Compartmentalized" prompts coming from Meta-LLaMA evaluators, using do_sample=False, simple format parse |

## Problems

There was a performance drop due to problems in constraining the inference model's responses to the correct format. This is the final form of the prompting function that uses a dictionary scheme similar to how OpenAI API functions consume their parameters.

```python
for i in tqdm(range(len(data))):
    total += 1
    item = data.iloc[i]
    final_prompts = item['input_final_prompts']
    correct_answer = item['input_correct_responses'][0]
    question = item['input_question']
    choices = item['input_choice_list']
    subject = item['subtask_name']

    messages = [
        {'role': 'system', 'content': final_prompts},
    ]

    outputs = pipe(messages, max_new_tokens=10, do_sample=False)
    
    output = outputs[0]['generated_text'][-1]['content'].strip()
    llama_answer = ''
    # ...
```

The recommendation is that instead of passing a `messages` dict, we can opt to pass in the raw message text directly to the inference model.

