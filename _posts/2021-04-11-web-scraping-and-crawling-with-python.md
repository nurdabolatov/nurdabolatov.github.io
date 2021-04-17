---
layout: post
title: Web Scraping and Crawling with Python
---

Have you ever wanted to download a website? For example, to keep a record of daily trending news for analysis. I will show you how to achieve that using different methods. Send me an email if you would like to contribute additional methods.

Let's start with a basic way:

```python
import requests

# get the website content
response = requests.get('http://example.com')

# save it on the disk
with open("example.html", "w") as file:
    file.write(response.text)
```

The above code will download the contents of `example.com` and store it in `example.html`. The script doesn't save CSS, JavasScript, and other static files, let alone sub-pages. This is a *naive* implementation of the "Save Page" feature in browsers. Fortunately, we can resolve these issues.

Before we jump to those solutions I want to share another common issue people face: authorization. In other words, a website requires particular cookie values:

```python
import requests

# open a session
with requests.session() as session:
    # set cookie values
    session.cookies.set('<session-name>', '<session-value>')

    response = session.get('http://example.com')
    with open("example.html", "w") as file:
        file.write(response.text)
```

In case you don't want to dig cookies, you can always authenticate with a simple post request:

```python
import requests

with requests.session() as session:
    # authenticate
    payload = {
        'action': 'login',
        'email': 'user@example.com',
        'password': 'WeakPassword',
    }
    session.post('http://example.com/login', data=payload)

    response = session.get('http://example.com')
    with open("example.html", "w") as file:
        file.write(response.text)
```

However, not all websites rely on cookies and you need to add other data in the web request (eg. request header data). But I digress.

## BeautifulSoup

Going back to downloading static files and crawling subpages. We will discuss the latter in a minute. Let's fix the former by scraping the content and explicitly saving the static files. Here we need some help from [beautifulsoup4](https://pypi.org/project/beautifulsoup4/):

```python
import requests
from bs4 import BeautifulSoup
from pathlib import Path

response = requests.get('http://example.com')

# get all <script /> elements
parsed_response = BeautifulSoup(response.text, 'html.parser')
scripts = parsed_response.find_all('script')

for script in scripts:
    # get "src" attribute value (eg. <script src="http://example.com/main.js" />)
    script_src = script.attrs.get('src')

    # handle the case when "src" is relative (eg. <script src="/main.js" />)
    if not script_src.startswith('http://') and not script_src.startswith('https://'):
        script_src = 'http://example.com' + script_src

    # download the script
    script_content = requests.get(script_src)

    # create folder to store the script (eg. http://example.com/nested/folders/main.js -> nested/folders)
    script_path = "/".join(script_src.split('/')[2:-1])
    Path(script_path).mkdir(parents=True, exist_ok=True)

    # get the name of the script
    script_name = script_src.split('/')[-1]

    # save the script
    full_path = script_path + '/' + script_name
    with open(full_path, 'w') as file:
        file.write(script_content.text)

    # update "src" attribute to load from disk
    script.attrs['src'] = './' + full_path

# save the page
with open('example.html', 'w') as file:
    file.write(str(parsed_response))
```

You can do the same for CSS, images, and other static files. 

As for crawling, we can scrape the links on the page and recursively perform the above:

```python
import requests
from bs4 import BeautifulSoup
from pathlib import Path


def crawl(url, depth=0):
    # do not go too deep when following links
    if depth > 1:
        print('Skipped: ' + url)
        return None

    response = requests.get(url)

    # get all <a /> elements
    parsed_response = BeautifulSoup(response.text, 'html.parser')
    links = parsed_response.find_all('a')

    for link in links:
        # get "href" attribute value
        href = link.attrs.get('href')

        # handle the case when "href" is relative (eg. <a href="/main" />)
        if not href.startswith('http://') and not href.startswith('https://'):
            href = 'http://example.com' + href

        # crawl "href" recursively
        path = crawl(href, depth + 1)

        # do not update "href" attribute if the page was skipped
        if path is None:
            continue

        # update "href" attribute to load from disk
        link.attrs['href'] = './' + path

    # ... save static files ...

    # construct a path to store the page
    path = '/'.join(url.split('/')[2:-1])
    Path(path).mkdir(parents=True, exist_ok=True)
    if path:
        path + '/'
    name = url.split('/')[-1] + '.html'
    full_path = path + name

    # save the page
    with open(full_path, 'w') as file:
        file.write(str(parsed_response))

    return full_path


print('Website saved at: ' + crawl('http://example.com'))
```

## Selenium

At this point, we have downloaded the entire website. But we are missing another important aspect: dynamically loaded content. Some websites don't load all elements on the first request and we must wait for additional content, like comments or video, to load before saving the page. Time for [Selenium](https://selenium-python.readthedocs.io/) to jump in and make our lives easier. Unlike BeautifulSoup, you must [install a web driver](https://selenium-python.readthedocs.io/installation.html#drivers) to use Selenium.

Selenium mimics user behavior by actually interacting with the website using a web browser (eg. Chrome, Edge, Firefox, Safari). It is an extremely useful tool for web crawling, end-to-end testing, and any automation which requires the browser.

A simple script to save a screenshot of the page using Selenium:

```python
from selenium import webdriver

driver = webdriver.Chrome()

driver.get('http://example.com')

driver.save_screenshot('example.png')

driver.close()
```

Neat. Right? Apart from saving the screenshot, there are tons of other ways to [interact with the page](https://selenium-python.readthedocs.io/navigating.html). Now let's see how can we take advantage of Selenium to save dynamic content.

```python
from selenium import webdriver, common
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions

driver = webdriver.Chrome()

driver.get('http://example.com')

# wait for a video to appear on the page
video = WebDriverWait(driver, 10).until(
    expected_conditions.presence_of_element_located((By.TAG_NAME, 'video'))
)

with open('example.html', 'w') as file:
    file.write(driver.page_source)

driver.close()
```

I am going to stop here but there are still many use cases I haven't covered. Also, note that my implementations don't cover edge cases such as skipping "nofollow" links and others. Feel free to reuse the code and add the necessary logic as needed. Send me an email if you would like to contribute to this tutorial.

Lastly, both BeautifulSoup and Selenium are powerful tools that you can use for quite a few purposes. You should use them with caution and be aware of the website's policies before downloading any data.