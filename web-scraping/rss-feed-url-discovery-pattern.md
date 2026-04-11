# RSS Feed URL Discovery Pattern

Many news sources and publications don't publicly document their RSS feed URLs. When you need to find them, systematic URL pattern testing is sometimes the only approach that works.

## Common Patterns to Try

Start with the domain root and try variations:

```
https://example.com/feed
https://example.com/feed.xml
https://example.com/rss
https://example.com/rss.xml
https://example.com/feeds/all.xml
https://example.com/atom.xml
https://example.com/index.xml
https://example.com/blog/feed
https://example.com/news/rss
https://example.com/api/v1/rss
```

## Automation Approach

```python
import requests

PATTERNS = [
    '/feed', '/feed.xml', '/rss', '/rss.xml',
    '/feeds/all.xml', '/atom.xml', '/index.xml',
    '/blog/feed', '/news/rss', '/api/v1/rss'
]

def discover_feed(base_url):
    for pattern in PATTERNS:
        url = base_url.rstrip('/') + pattern
        try:
            response = requests.get(url, timeout=10)
            if response.status_code == 200:
                content_type = response.headers.get('content-type', '')
                if 'xml' in content_type or 'rss' in response.text[:1000]:
                    return url
        except requests.RequestException:
            continue
    return None
```

## When This Is Necessary

Some major publications simply don't publish their RSS URLs anywhere - not in page source, not in documentation, not in robots.txt. You only find them through trial and error or by asking someone who already knows.

## The Lesson

Don't give up after checking the obvious places. RSS feeds often exist but are intentionally hidden (or just forgotten). A systematic 15-20 URL pattern test takes minutes and can unlock data sources you'd otherwise miss.
