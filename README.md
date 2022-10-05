# as-scraper

[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/as-scraper.svg)](https://pypi.org/project/as-scraper/)
[![PyPI - Downloads](https://img.shields.io/pypi/dm/as-scraper)](https://pypi.org/project/as-scraper/)

Python library for scraping using Selenium

> If you are looking for the library implemented inside airflow, go to https://github.com/Avila-Systems/as-scraper-airflow.

# Installation

The **as-scraper** library uses Geckodriver (Firefox) for scraping with the Selenium library.
In order to use it, you need to have an Geckodriver dependency. Check the [selenium documentation](https://www.selenium.dev/documentation/webdriver/getting_started/install_drivers/) for details about how to install the Firefox browser driver.

# Usage

## Creating a simple scraper

Lets say that we want to scrap [yellowpages.com](https://www.yellowpages.com). Our target data would be the popular cities that we can find in the [sitemap](https://www.yellowpages.com/sitemap) url.

Our output data will have two columns: `name` of the city and `url` which is linked to the city. For example, for *Houston*, we would want the following output:

| name | url |
|:-----|:----|
|Houston|https://www.yellowpages.com/houston-tx|

### Declaring our Scraper Class

So first we create a scraper that extends from the Scraper class, and define the `COLUMNS` variable to `['name', 'url']`.

Create the *scrapers/yellowpages.py* file and type the following code into it:

```python
from as_scraper.scraper import Scraper


class YellowPagesScraper(Scraper):
    COLUMNS = ['name', 'url']

```

### Deciding wether to load javascript or not

Now, there are two execution options when running scrapers. We can either *load javascript* which uses the **Selenium** library, or not load javascript and use the *requests* library for http requests.

For this example, let's go ahead and use the **Selenium** library. To configure this, simply add the following variable to your scraper:

```python
from as_scraper.scraper import Scraper


class YellowPagesScraper(Scraper):
    COLUMNS = ['name', 'url']
    LOAD_JAVASCRIPT = True

```

### Defining the `scrape_handler`

And the magic comes in the next step. We will define the `scrape_handler` method in our class, which will have the responsibility to scrape a given url and extract the data from it.

> All scrapers must define the `scrape_handler` method.

```python
from typing import Optional
from selenium.webdriver import Firefox
from selenium.webdriver.common.by import By
import pandas as pd
from as_scraper.scraper import Scraper


class YellowPagesScraper(Scraper):
    COLUMNS = ['name', 'url']
    LOAD_JAVASCRIPT = True

    def scrape_handler(self, url: str, html: Optional[str] = None, driver: Optional[Firefox] = None, **kwargs) -> pd.DataFrame:
        rows = []
        div_tag = driver.find_element(By.CLASS_NAME, "row-content")
        div_tag = div_tag.find_element(By.CLASS_NAME, "row")
        section_tags = div_tag.find_elements(By.TAG_NAME, "section")
        for section_tag in section_tags:
            a_tags = section_tag.find_elements(By.TAG_NAME, "a")
            for a_tag in a_tags:
                city_name = a_tag.text
                city_url = a_tag.get_attribute("href")
                rows.append({"name": city_name, "url": city_url})
        df = pd.DataFrame(rows, columns=self.COLUMNS)
        return df

```

### Execution

Finally, to execute the scraper you must call the **execute* method.

```python
from typing import Optional
from selenium.webdriver import Firefox
from selenium.webdriver.common.by import By
import pandas as pd
from as_scraper.scraper import Scraper

class YellowPagesScraper(Scraper):
    COLUMNS = ['name', 'url']
    LOAD_JAVASCRIPT = True

    def scrape_handler(self, url: str, html: Optional[str] = None, driver: Optional[Firefox] = None, **kwargs) -> pd.DataFrame:
        rows = []
        div_tag = driver.find_element(By.CLASS_NAME, "row-content")
        div_tag = div_tag.find_element(By.CLASS_NAME, "row")
        section_tags = div_tag.find_elements(By.TAG_NAME, "section")
        for section_tag in section_tags:
            a_tags = section_tag.find_elements(By.TAG_NAME, "a")
            for a_tag in a_tags:
                city_name = a_tag.text
                city_url = a_tag.get_attribute("href")
                rows.append({"name": city_name, "url": city_url})
        df = pd.DataFrame(rows, columns=self.COLUMNS)
        return df

if __name__ == '__main__':
    urls = ['https://www.yellowpages.com/sitemap']
    scraper = YellowPagesScraper(urls)
    results, errors = scraper.execute()
    print(results)
    print(errors)

```

Now go ahead and run `python scrapers/yellowpages.py`. Have fun!
