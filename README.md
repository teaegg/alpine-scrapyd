alpine-scrapyd
=======

![](http://dockeri.co/image/teaegg/alpine-scrapyd)

[Scrapy][1] is an open source and collaborative framework for extracting the
data you need from websites. In a fast, simple, yet extensible way.

[Scrapyd][2] is a service for running Scrapy spiders.  It allows you to deploy
your Scrapy projects and control their spiders using a HTTP JSON API.

[Scrapyd-client][3] is a client for scrapyd. It provides the scrapyd-deploy
utility which allows you to deploy your project to a Scrapyd server.

[ScrapyJS][4] provides Scrapy+JavaScript integration using Splash.

This image is based on `alpine` without any useless packages installed.
Only 4 latest python packages are installed:

- `scrapy`: git+https://github.com/scrapy/scrapy.git
- `scrapyd`: git+https://github.com/scrapy/scrapyd.git
- `scrapyd-client`: git+https://github.com/scrapy/scrapyd-client.git
- `scrapyjs`: git+https://github.com/scrapinghub/scrapy-splash.git

Please use this image as base for your own project.

## Run it as background-daemon for scrapyd

```sh
docker run -d --restart always --name crawler -p 6800:6800 teaegg/python-crawler
firefox http://localhost:6800
```

## Run it as interactive-shell for scrapy

```sh
$ cat > stackoverflow_spider.py
```

```python
import scrapy

class StackOverflowSpider(scrapy.Spider):
    name = 'stackoverflow'
    start_urls = ['http://stackoverflow.com/questions?sort=votes']

    def parse(self, response):
        for href in response.css('.question-summary h3 a::attr(href)'):
            full_url = response.urljoin(href.extract())
            yield scrapy.Request(full_url, callback=self.parse_question)

    def parse_question(self, response):
        yield {
            'title': response.css('h1 a::text').extract()[0],
            'votes': response.css('.question .vote-count-post::text').extract()[0],
            'body': response.css('.question .post-text').extract()[0],
            'tags': response.css('.question .post-tag::text').extract(),
            'link': response.url,
        }
```

```sh
docker run -it --rm -v `pwd`:/code -w /code teaegg/python-crawler sh
scrapy runspider stackoverflow_spider.py -o top-stackoverflow-questions.json
cat top-stackoverflow-questions.json
exit
```

[1]: https://github.com/scrapy/scrapy
[2]: https://github.com/scrapy/scrapyd
[3]: https://github.com/scrapy/scrapyd-client
[4]: https://github.com/scrapinghub/scrapy-splash
