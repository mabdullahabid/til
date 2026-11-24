# Move CPU-Bound Fallbacks Off the Async Event Loop

An `async` function is not automatically non-blocking. If a fallback parser does synchronous CPU work inside it, the event loop cannot serve other coroutines until the parser returns.

```python
import asyncio


async def extract_page(page):
    text = await fetch_pruned_text(page)
    if text:
        return text

    return await asyncio.to_thread(parse_raw_html, page.html)
```

Use this for synchronous fallback work when the surrounding crawl needs to keep handling network and browser tasks. `to_thread()` moves the blocking call to a worker thread, freeing the event loop; it does not make the parser itself faster. For heavy pure-Python CPU work, the GIL may prevent parallel execution, so benchmark and consider a process pool or separate worker if throughput matters.
