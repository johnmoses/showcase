# Custom Transformer Fine-Tuning — Built Without AI Assistance (2024)

**System:** ChurchReports (predecessor to Life Reports)  
**Problem:** Church reporters needed to interact with the platform via natural language chat — querying attendance, submitting reports, getting help — without typing structured commands.

## What Was Built

Three custom fine-tuned models, trained locally on a MacBook, deployed to production:

| Model | Architecture | Task | Training |
|-------|-------------|------|----------|
| **Collator** | BERT (bert-base-uncased) | Intent classification (44 classes) | 100 epochs, 500 steps |
| **SQL** | T5-small | Natural language → SQL | 17,600 samples, 2,200 steps (full) + 10,560 steps (PEFT) |
| **Support** | T5-small | Question answering | Fine-tuned on domain QA pairs |

## Model 1: BERT Intent Classifier (44 intents)

Classifies user messages into domain-specific actions: `report_submission`, `fellowship_attendance_list`, `location_admin_add`, `finance_list`, etc.

```python
# Training pipeline — BERT fine-tuned for 44-class intent classification
model_name = "bert-base-uncased"

# 189 training samples across 44 intent classes (small dataset, high accuracy)
model = BertForSequenceClassification.from_pretrained(
    model_name, num_labels=44, id2label=id2label, label2id=label2id
)

training_args = TrainingArguments(
    output_dir='./output',
    num_train_epochs=100,
    per_device_train_batch_size=32,
    warmup_steps=100,
    weight_decay=0.05,
    load_best_model_at_end=True
)

trainer = Trainer(model=model, args=training_args,
                  train_dataset=train_dataloader,
                  eval_dataset=test_dataloader,
                  compute_metrics=compute_metrics)
trainer.train()  # 3 min 9s on CPU
```

**Results:**
| Split | Accuracy | F1 | Precision | Recall |
|-------|----------|-----|-----------|--------|
| Train | 98.6% | 96.2% | 96.5% | 96.4% |
| Test | 87.5% | 70.7% | 70.0% | 72.0% |

**Inference — deployed via HuggingFace pipeline:**
```python
pipe = pipeline("sentiment-analysis", model=model, tokenizer=tokenizer)
pipe("Hello")  # → {'label': 'greeting', 'score': 0.9953}

# Production routing: score < 0.8 → "Don't understand", else → execute intent
def model_response(text):
    score = pipe(text)[0]['score']
    tag = pipe(text)[0]['label']
    if score < 0.8:
        return "Don't understand!"
    label = model.config.label2id[tag]
    return random.choice(intents['intents'][label]['responses']), tag
```

## Model 2: T5 Text-to-SQL (Full Fine-Tune + PEFT/LoRA)

Converts natural language questions into SQL queries against the platform's schema.

```python
# Dataset: 17,600 train samples merged from 3 sources
# - b-mc2/sql-create-context (8K general SQL)
# - Clinton/Text-to-sql-v1 (8K general SQL)
# - Local domain data (1,600 platform-specific queries)

def tokenize_function(example):
    prompt = [f"Tables:\n{ctx}\n\nQuestion:\n{q}\n\nAnswer:\n"
              for ctx, q in zip(example['context'], example['question'])]
    example['input_ids'] = tokenizer(prompt, padding="max_length",
                                      truncation=True, return_tensors="pt").input_ids
    example['labels'] = tokenizer(example['answer'], padding="max_length",
                                   truncation=True, return_tensors="pt").input_ids
    return example

# Full fine-tune: 2 epochs, 2h 13min on CPU
trainer.train()  # eval_loss: 1.44 → 0.024 (58x improvement)

# PEFT/LoRA: 3 epochs, 6h on CPU — only 1.9% params trainable
peft_config = LoraConfig(
    r=32, lora_alpha=32, lora_dropout=0.05,
    bias="none", task_type=TaskType.SEQ_2_SEQ_LM
)
# trainable params: 1,179,648 || all params: 61,686,272 || trainable%: 1.9123
```

**ROUGE evaluation (fine-tuned vs base):**
| Model | ROUGE-1 | ROUGE-2 | ROUGE-L |
|-------|---------|---------|---------|
| Base T5 (zero-shot) | 0.061 | 0.029 | 0.058 |
| **Fine-tuned T5** | **0.976** | **0.933** | **0.961** |

**Before fine-tuning:**
```
Input:  "Which round has pain and glory 2006 as the event?"
Output: "Welche Runde hat Schmerz und Ruhm 2006 als Veranstaltung?" (German translation!)
```

**After fine-tuning:**
```
Input:  "Which round has pain and glory 2006 as the event?"
Output: SELECT round FROM table_name_94 WHERE event = "pain and glory 2006" ✓
```

## Deployment

All 3 models published to HuggingFace Hub (`johnmose/crp-base-model`, `johnmose/crp-sql`, `johnmose/crp-support`) and loaded in the Flask SocketIO server for real-time chat inference.

## Why This Matters

- Fine-tuned transformers from scratch **before ChatGPT existed as a coding tool**
- Understood BERT vs T5 architecture tradeoffs (encoder-only for classification, encoder-decoder for generation)
- Applied PEFT/LoRA for parameter-efficient training on consumer hardware
- Built the full pipeline: data prep → training → evaluation → deployment → production inference
- Merged domain-specific data with public datasets for transfer learning
