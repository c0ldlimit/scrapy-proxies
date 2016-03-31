Based on https://github.com/aivarsk/scrapy-proxies

Random proxy with retries middleware for Scrapy (http://scrapy.org/)
--------------------------------------------------------------------

Processes Scrapy requests using a random proxy from list to avoid IP ban and
improve crawling speed.

After RETRY_TIMES fails the RandomProxyRetryMiddleware class trying to use a new proxy, rather than turn off a spider.

middlewares.py
--------------
Put RandomProxyRetryMiddleware class into your middlewares.py file.

settings.py
-----------

    # Retry many times since proxies often fail
    RETRY_TIMES = 2
    # Unless you want to wait around
    DOWNLOAD_TIMEOUT = 4

    # Retry on most error codes since proxies fail for different reasons
    RETRY_HTTP_CODES = [500, 503, 504, 400, 403, 404, 408]

    DOWNLOADER_MIDDLEWARES = {
        'scrapy.downloadermiddlewares.retry.RetryMiddleware': None,
        # FIXME: use your project name
        'yourproject.middlewares.RandomProxyRetryMiddleware': 100,
        'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 110,
    }

    # Proxy list containing entries like
    # http://host1:port
    # http://username:password@host2:port
    # http://host3:port
    # ...
    PROXY_LIST = 'yourproject/proxylist.txt'


Your spider
-----------

In each callback ensure that proxy /really/ returned your target page by
checking for site logo or some other significant element.
If not - retry request with dont_filter=True

    if not hxs.select('//get/site/logo'):
        yield Request(url=response.url, dont_filter=True)
