# Scrapy with selenium4
[![PyPI](https://img.shields.io/pypi/v/scrapy-selenium4.svg)](https://pypi.python.org/pypi/scrapy-selenium4)

Scrapy middleware to handle javascript pages using selenium >= 4.0.0.

## Installation
```
$ pip install scrapy-selenium4
```
You should use **python>=3.6**.
You will also need one of the Selenium [compatible browsers](http://www.seleniumhq.org/about/platforms.jsp).

## Configuration
Add the browser to use, the path to the driver executable, and the arguments to pass to the executable to the scrapy settings.py:
```python
# Add the `SeleniumMiddleware` to the downloader middlewares
DOWNLOADER_MIDDLEWARES = {
    'scrapy_selenium4.SeleniumMiddleware': 800
}
```
Other configurations(Default):
```python
SELENIUM_DRIVER_NAME = 'chrome'
from shutil import which
SELENIUM_DRIVER_EXECUTABLE_PATH = which('chromedriver')
SELENIUM_DRIVER_ARGUMENTS=[
    '--headless=new',
    '--no-sandbox',
    '--disable-gpu',
    '--window-size=1280,1696',
    '--disable-blink-features',
    '--disable-blink-features=AutomationControlled',
    '--user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"',
]
# In order to use a remote Selenium driver, specify SELENIUM_COMMAND_EXECUTOR instead of SELENIUM_DRIVER_EXECUTABLE_PATH.
# SELENIUM_COMMAND_EXECUTOR = 'http://localhost:4444/wd/hub'
```
## Usage
Use the `scrapy_selenium4.SeleniumRequest` instead of the scrapy built-in `Request` like below:
```python
from scrapy_selenium4 import SeleniumRequest

    def start_requests(self):
        for url in start_urls:
            yield SeleniumRequest(url=url, callback=self.parse_result)
```
The request will be handled by selenium, and the request will have an additional `meta` key, named `driver` containing the selenium driver with the request processed.
```python
    def parse_result(self, response):
        print(response.request.meta['driver'].title)
```
For more information about the available driver methods and attributes, refer to the [selenium python documentation](http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webdriver)

The `selector` response attribute work as usual (but contains the html processed by the selenium driver).
```python
def parse_result(self, response):
    print(response.selector.xpath('//title/@text'))
```

### Additional arguments
The `scrapy_selenium4.SeleniumRequest` accept 4 additional arguments:

#### `wait_time` / `wait_until`

When used, selenium will perform an [Explicit wait](http://selenium-python.readthedocs.io/waits.html#explicit-waits) before returning the response to the spider.
```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC

yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    wait_time=10,
    wait_until=EC.element_to_be_clickable((By.ID, 'someid'))
)
```

#### `screenshot`
When used, selenium will take a screenshot of the page and the binary data of the .png captured will be added to the response `meta`:
```python
yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    screenshot=True
)

def parse_result(self, response):
    with open('image.png', 'wb') as image_file:
        image_file.write(response.meta['screenshot'])
```

New way to screenshot:
```python
yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    screenshot=f'image.png'
)

def parse_result(self, response):
    pass
```

#### `script`
When used, selenium will execute custom JavaScript code after page loaded.
```python
yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    script='window.scrollTo(0, document.body.scrollHeight);',
)
```

### `scroll_bottom`
When used, selenium will scroll to bottom.
```python
yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    scroll_bottom=True
)
```
