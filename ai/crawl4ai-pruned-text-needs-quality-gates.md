# Crawl4AI Pruned Text Needs Quality Gates

Non-empty pruned text is not necessarily the page you wanted. A content pruner can return only a cookie banner or a tiny fragment while the raw markdown contains the article.

Treat pruning as a candidate, not an unconditional replacement:

```python
def looks_like_consent_banner(text):
    phrases = ("accept cookies", "manage preferences", "cookie settings")
    return any(phrase in text.lower() for phrase in phrases)


def usable(text, raw, *, min_chars=200, min_ratio=0.05):
    text = (text or "").strip()
    raw = (raw or "").strip()
    if not text or looks_like_consent_banner(text):
        return False
    return len(text) >= min_chars or (
        bool(raw) and len(text) / len(raw) >= min_ratio
    )

def select_content(pruned_text, main_text, raw_markdown):
    if usable(pruned_text, raw_markdown):
        return pruned_text
    if usable(main_text, raw_markdown):
        return main_text
    return raw_markdown
```

The banner detector is illustrative and not a built-in Crawl4AI function. The numbers are illustrative: tune them against real pages, including legitimately short pages. Compare both size and proportion, and reject banner-only results even if they pass a length threshold. A secondary main-text extraction needs the same checks before it can replace the raw page.
