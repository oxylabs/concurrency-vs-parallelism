# Concurrency vs. Parallelism

[![](https://dcbadge.vercel.app/api/server/eWsVUJrnG5)](https://discord.gg/GbxmdGhZjq)

[![Oxylabs promo code](https://raw.githubusercontent.com/oxylabs/product-integrations/refs/heads/master/Affiliate-Universal-1090x275.png)](https://oxylabs.go2cloud.org/aff_c?offer_id=7&aff_id=877&url_id=112)


[<img src="https://img.shields.io/static/v1?label=&message=Concurrency+vs+Parallelism&color=brightgreen" />](https://github.com/topics/Concurrency-vs-Parallelism)

- [What is concurrency?](#what-is-concurrency)
- [What is a thread?](#what-is-a-thread)
- [Practical example](#practical-example)
- [Using concurrency to speed up processes](#using-concurrency-to-speed-up-processes)
- [What is parallelism?](#what-is-parallelism)
- [Using parallelism to speed up processes](#using-parallelism-to-speed-up-processes)

This article gives you an overview of the differences between concurrency and parallelism. 

This article gives you an overview of cURL.

For a detailed explanation, see our [blog post](https://oxylabs.io/blog/concurrency-vs-parallelism).

## What is concurrency?

Concurrency is pausing and resuming threads.

This capability of modern CPUs to pause and resume tasks so fast gives an illusion that the tasks are running in parallel. **However, this is not parallel. This is concurrent.**

Concurrency can be broadly understood as multi-threading. There are usually many ways of creating concurrent applications, and threading is just one of them. 

### What is a thread?

In broad terms, a thread is the smallest set of tasks that can be handled and managed by the operating system without any dependencies on each other. 

Python provides a powerful threading module for creating and managing threads. 

## Practical example

To understand how concurrency works, let’s solve a practical problem. The task is to process over 200 pages as fast as possible. Here are the details:

**Step 1.** Go to the Wikipedia page with [a list of countries by population](https://en.wikipedia.org/wiki/List_of_countries_by_population_(United_Nations)) and get the links of all the 233 countries listed on this page.

**Step 2.** Go to all these 233 pages and save the HTML locally.

Let’s create a function to get all the links. At first, we won’t involve concurrency or parallelism here.

```python
import requests
from bs4 import BeautifulSoup
from urllib.parse import urljoin

def get_links():
    countries_list = 'https://en.wikipedia.org/wiki/List_of_countries_by_population_(United_Nations)'
    all_links = []
    response = requests.get(countries_list)
    soup = BeautifulSoup(response.text, "lxml")
    countries_el = soup.select('td .flagicon+ a')
    for link_el in countries_el:
        link = link_el.get("href")
        link = urljoin(countries_list, link)
        all_links.append(link)
    return all_links
```

Create a function to fetch and save a link.

```python
def fetch(link):
    response = requests.get(link)
    with open(link.split("/")[-1]+".html", "wb") as f:
        f.write(response.content)
```

Finally, let’s call this function in a loop:

```python
import time

if __name__ == '__main__':
    links = get_links()
    print(f"Total pages: {len(links)}")
    start_time = time.time()
    # This for loop will be optimized
    for link in links:
        fetch(link)

    duration = time.time() - start_time
    print(f"Downloaded {len(links)} links in {duration} seconds")
```

With our computer, this took **137.37 seconds.** Our objective is to bring this time down.

### Using concurrency to speed up processes

Although we can create threads manually, we’ll have to start them manually and call the join method on each thread so that the main program waits for all these threads to complete.

The better approach is to use the `ThreadPoolExecutor` class. 

First, we need to import ThreadPoolExecutor:

```
from concurrent.futures import ThreadPoolExecutor
```

Now, the for loop written above can be changed to the following.

```
with ThreadPoolExecutor(max_workers=16) as executor:
      executor.map(fetch, links)
```

The final result is astonishing! All these 233 links were downloaded in **11.23 seconds**. 

## What is parallelism?

This is a type of computation in which multiple processors carry out many processes simultaneously. 

Parallelism is multiple threads running on multiple CPUs.

### Using parallelism to speed up processes

Let’s start with importing the required module:

```python
from multiprocessing import Pool, cpu_count
```

Now we can replace the for loop in the synchronous code with this code:

```python
with Pool(cpu_count()) as p:
        p.map(fetch, links)
```

This fetches all 233 links in **18.10 seconds**. It’s also noticeably faster than the synchronous version which took around 138 seconds.

If you wish to learn more about Concurrency vs. Parallelism, see our [blog post](https://oxylabs.io/blog/concurrency-vs-parallelism).
