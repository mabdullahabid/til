# Chatbot Container Validation Prompt Strategy

For validating user inputs like container sizes, use explicit enumeration of valid options rather than open-ended validation.

## The Problem

Users were asking for "45ft containers" which the business didn't sell. The chatbot would accept this, collect all other required information, then fail when trying to process the invalid size.

## The Fix

Explicitly list valid sizes in the prompt and validate before collecting additional data:

```python
CONTAINER_SIZES = ["10ft", "20ft", "40ft"]

SYSTEM_PROMPT = """
You are a container sales assistant.

Valid container sizes: 10ft, 20ft, 40ft.

Before collecting any other information, you MUST:
1. Ask the user what size they need
2. Validate the size is one of: 10ft, 20ft, 40ft
3. If they request an invalid size (like 45ft), politely inform them 
   of available sizes and ask again
4. Only proceed to collect name, location, etc. AFTER valid size is confirmed

Never collect partial data for invalid sizes.
"""
```

## Why This Matters

Stricter validation at the start prevents:
- Wasted conversation turns collecting unusable data
- User frustration from being told "actually, we can't do that" after answering multiple questions
- Backend errors from invalid data reaching the processing stage

## The Lesson

Validate early, validate explicitly. Don't assume the LLM will infer valid ranges from context - tell it exactly what's allowed and enforce it before moving forward.
