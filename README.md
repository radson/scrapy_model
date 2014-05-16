Create scraper using Scrapy Selectors
============================================

allows you to select by CSS or by XPATH

Implemented in a Model approach, you create a Fetcher class and defines some fields which points to Xpath or Css selectors, those fields are fetched and an object populated with data.

Data can be normalized using ``parse_<field>`` methods.

### Instalation

easy to install 

If running ubuntu maybe you need to run ``sudo apt-get install python-scrapy``

```
git clone https://github.com/rochacbruno/scrapy_model
cd scrapy_model
pip install -r requirements.txt
python example.py
```
> NOTE: I need to create a setup.py and release to PyPI  
> Take a look at the example.py


Example code to fetch the url http://en.m.wikipedia.org/wiki/Guido_van_Rossum

```
#coding: utf-8

from scrapy_model import BaseFetcherModel, CSSField, XPathField


class TestFetcher(BaseFetcherModel):
    photo_url = XPathField('//*[@id="content"]/div[1]/table/tr[2]/td/a')

    nationality = CSSField(
        '#content > div:nth-child(1) > table > tr:nth-child(4) > td > a',
    )

    links = CSSField(
        '#content > div:nth-child(11) > ul > li > a.external::attr(href)',
        auto_extract=True
    )

    def parse_photo_url(self, selector):
        return "http://en.m.wikipedia.org/{}".format(
            selector.xpath("@href").extract()[0]
        )

    def parse_nationality(self, selector):
        return selector.css("::text").extract()[0]

    def parse_name(self, selector):
        return selector.extract()[0]

    def post_parse(self):
        # executed after all parsers
        # you can load any data on to self._data
        # access self._data and self._fields for current data
        # self.selector contains original page
        # self.fetch() returns original html
        self._data.url = self.url


class DummyModel(object):
    """
    For tests only, it can be a model in your database ORM
    """


if __name__ == "__main__":
    from pprint import pprint

    fetcher = TestFetcher(cache_fetch=True)
    fetcher.url = "http://en.m.wikipedia.org/wiki/Guido_van_Rossum"

    # Mappings can be loaded from a json file
    # fetcher.load_mappings_from_file('path/to/file')
    fetcher.mappings['name'] = {
        "css": ("#section_0::text")
    }

    fetcher.parse()

    print "Fetcher holds the data"
    print fetcher._data.name
    print fetcher._data

    # How to populate an object
    print "Populating an object"
    dummy = DummyModel()

    fetcher.populate(dummy, fields=["name", "nationality"])
    # fields attr is optional
    print dummy.nationality
    pprint(dummy.__dict__)

```

# outputs


```
Fetcher holds the data
Guido van Rossum
{'links': [u'http://www.python.org/~guido/',
           u'http://neopythonic.blogspot.com/',
           u'http://www.artima.com/weblogs/index.jsp?blogger=guido',
           u'http://python-history.blogspot.com/',
           u'http://www.python.org/doc/essays/cp4e.html',
           u'http://www.twit.tv/floss11',
           u'http://www.computerworld.com.au/index.php/id;66665771',
           u'http://www.stanford.edu/class/ee380/Abstracts/081105.html',
           u'http://stanford-online.stanford.edu/courses/ee380/081105-ee380-300.asx'],
 'name': u'Guido van Rossum',
 'nationality': u'Dutch',
 'photo_url': 'http://en.m.wikipedia.org//wiki/File:Guido_van_Rossum_OSCON_2006.jpg',
 'url': 'http://en.m.wikipedia.org/wiki/Guido_van_Rossum'}
Populating an object
Dutch
{'name': u'Guido van Rossum', 'nationality': u'Dutch'}
```
