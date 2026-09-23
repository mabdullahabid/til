# QuerySet.update() Bypasses Model and Wagtail Signals

Django's `QuerySet.update()` writes directly to the database. It does not call `Model.save()` and therefore does not emit lifecycle events such as Wagtail's `page_published` signal. If cache revalidation or indexing depends on that event, use the normal publish path or trigger the side effect explicitly.

```python
Page.objects.filter(pk=page.pk).update(seo_title=title)
# The database row changes, but page_published is not emitted.
```
