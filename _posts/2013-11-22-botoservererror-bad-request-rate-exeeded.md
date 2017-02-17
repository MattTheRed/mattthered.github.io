---
layout: post
title: BotoServerError - 400 Bad Request - Rate Exceeded
redirect_from:
  - /post/67763258327/botoservererror-400-bad-request-rate-exceeded
---
This is for anyone out there using [python-boto](http://t.umblr.com/redirect?z=https%3A%2F%2Fgithub.com%2Fboto%2Fboto&t=ZDk0MWRiOTJkYzI2MDliYjQzODgyYTdjNjFkN2ZlNDdlMzMzNDcyYSxkM09zMzYzYg%3D%3D&b=t%3AV3QsY1F6pCug4-HUnFvSyw&p=http%3A%2F%2Fwww.mattthered.com%2Fpost%2F67763258327%2Fbotoservererror-400-bad-request-rate-exceeded&m=1) with [Amazon CloudSearch](http://t.umblr.com/redirect?z=http%3A%2F%2Faws.amazon.com%2Fcloudsearch%2F&t=ZDE4Mzk3YzczOWE1N2FjZGEyODkzMWFhZWE5ZTJjOTRkYjg1OTNhNixkM09zMzYzYg%3D%3D&b=t%3AV3QsY1F6pCug4-HUnFvSyw&p=http%3A%2F%2Fwww.mattthered.com%2Fpost%2F67763258327%2Fbotoservererror-400-bad-request-rate-exceeded&m=1), who is getting a throttle limit and googling around for an answer. You’re probably getting an error like this:
```
<ErrorResponse xmlns=“http://cloudsearch.amazonaws.com/doc/2011-02-01/”>
  <Error>
    <Type>Sender</Type>
    <Code>Throttling</Code>
    <Message>Rate exceeded</Message>
  </Error>
  <RequestId>c753f49e-26b9-11e3-a879-0d3b8f7ade79</RequestId>
</ErrorResponse>
```

Here’s what’s happening: Although there aren’t any limits on the volume of searches you can make with CloudSearch, there are throttling limits on the preliminary calls to make to get the document & search service.

This can be solved by caching the search service object. Here’s an example with Django’s caching:

```python
import boto
django_cache

cs_conn = boto.connect_cloudsearch(
    aws_access_key_id=YOUR_KEY_ID,
    aws_secret_access_key=YOUR_ACCESS_KEY,
)

search_kwargs = {}
search_kwargs['q'] = "Stuff to find"

try:
    search_service = django_cache.get("cloudsearch_search_service")
    # Need to try to execute something for it to fail         
    results = search_service.search(**search_kwargs)
except:
    domain = cs_conn.lookup("domain-name")
    document_service = domain.get_document_service()
    search_service = document_service.get_search_service()
    django_cache.set("cloudsearch_search_service", search_service, 300)
    results = search_service.search(**search_kwargs)

print results.doc


```

