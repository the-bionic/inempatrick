---
title: Web scraping with Node.js
date: "2019-04-03T23:46:37.121Z"
template: "post"
draft: false
slug: "/posts/perfecting-the-art-of-perfection/"
category: "Design Inspiration"
tags:
  - "Handwriting"
  - "Learning to write"
description: "Web scraping is a technique used for retrieving data from websites. You fetch the page’s contents, and then extract the data you need from the page for processing, saving it, or simply displaying it on your app. It comes in handy when the app/website you are trying to scrape does not expose any external API for public consumption. It is worth noting that some sites do not allow scraping, so be aware of that before you attempt it."
---

![Nulla faucibus vestibulum eros in tempus. Vestibulum tempor imperdiet velit nec dapibus](/media/image-2.jpg)

Web scraping is a technique used for retrieving data from websites. You fetch the page’s contents, and then extract the data you need from the page for processing, saving it, or simply displaying it on your app. It comes in handy when the app/website you are trying to scrape does not expose any external API for public consumption. It is worth noting that some sites do not allow scraping, so be aware of that before you attempt it.

In Node.js, we can fetch the webpage using an HTTP client, like axios and use cheerio for extracting the data we need from the page.
Here, I will walk us through the process of:
Fetching a webpage
Extracting data from a webpage
Displaying the content on a webpage
Saving the data in JSON format
Fetching the webpage
The site we will be scraping is remoteok. It is a job board where remote work is listed, with tags, company names, and categories.
We are using axios for data fetching, but first, we install our dependencies

``` bash
$ mkdir scraper && cd scrapper
$ npm init -y
$ npm install --save axios cheerio
```

And use it like so:

``` js
index.js
const siteUrl = "https://remoteok.io/";
const axios = require("axios");
const fetchData = async () => {
  const result = await axios.get(siteUrl);
  return cheerio.load(result.data);
};
```

## Extracting data from a webpage
Using the helper function we created from the previous section, we can fetch the webpage, and using cheerio library, it provides us with utility functions to manipulate the page easily.

Open the developer tools on your favorite browser and inspect the element you want to target. On Google Chrome, you can open the developer tools with ctrl + shift + i and you will see the markup on the Elements tab. You can also easily right click the element on the webpage and click inspect.
We are going to target the Post a Job button and log the result to the console.

We are going to target the Post a Job button and log the result to the console.

``` js
const $ = await fetchData();
const postJobButton = $('.top > .action-post-job').text();
console.log(postJobButton) // Logs 'Post a Job'
```

We have successfully crawled remoteok and extracted data from it.
Let’s do something more ambitious. We are going to get all the tags, company names, locations, and positions so we can display that on a webpage. We will serve the webpage using Express. I have the finished app here on my github repo
Setup

``` bash
// continuing from the previous example
$ npx express-generator //Scaffold an express app. Follow the command line instructions. 
$ npm install
```

We will add this line at the bottom of app.js to start the server

``` js
app.js
const port = 9000
app.listen(port, () => console.log(`App listening on port ${port}!`))
Add a scrapper.js file on the project root, where we will put in the scrapper logic
scraper.js
const cheerio = require("cheerio");
const axios = require("axios");
const siteUrl = "https://remoteok.io/";
let siteName = "";
const categories = new Set();
const tags = new Set();
const locations = new Set();
const positions = new Set();
```

We are creating sets to hold our data because there is a possibility of getting duplicate categories, tags, locations or positions from the webpage. Set() is a data structure that does not allow duplicates.
Moving on…

``` js
scraper.js
const fetchData = async () => {
  const result = await axios.get(siteUrl);
  return cheerio.load(result.data);
};
const getResults = async () => {
  const $ = await fetchData();
  siteName = $('.top > .action-post-job').text();
  $(".tags .tag").each((index, element) => {
    tags.add($(element).text());
  });
  $(".location").each((index, element) => {
   locations.add($(element).text());
  });
  $("div.nav p").each((index, element) => {
   categories.add($(element).text());
  });

 $('.company_and_position [itemprop="title"]')
  .each((index, element) => {
  positions.add($(element).text());
 });
//Convert to an array so that we can sort the results.
return {
  positions: [...positions].sort(),
  tags: [...tags].sort(),
  locations: [...locations].sort(),
  categories: [...categories].sort(),
  siteName,
 };
};
module.exports = getResults;
```

Displaying the contents on a webpage

``` js
routes/index.js
const express = require("express");
const router = express.Router();
const getResults = require("../scraper");
/* GET home page. */
router.get("/", async function(req, res, next) {
  const result = await getResults();
  res.render("index", result);
});
module.exports = router;
```

```
views/index.jade
extends layout
block content
 h1 #{siteName}!
 h3 Categories
 ul
  each val in categories
   li= val
 h3 Positions
 ul
  each val in positions
   li= val
 h3 Locations
 ul
 each val in locations
  li= val
 h3 Tags
 ul
  each val in tags
   li= val
```

Start the node server:

``` bash
$ node app.js
```
Visit port localhost:9000 to see app.

Saving the data in JSON format
We can also save our extracted data in a JSON file if we want, with a little amount of code.

``` js
utils/converter.js
const fs = require('fs');
const getResults = require('../scraper');
(async () => {
  let results = await getResults()
  let jsonString = JSON.stringify(results);
  fs.writeFileSync('../output.json', jsonString, 'utf-8');
})()
```

This will create an output.json file in the root of the application with the data.

## Caution
It is worth mentioning that scraping scripts can break easily because you do not control the structure of the site that is being scraped. Even if you do, you’d still need to be extra vigilant not to break the script. Also, some websites do not allow scraping. Make sure you do your research before attempting to scrape any website.

## Conclusion
Scraping a website in Node.js is very easy with cheerio, as we have seen. Get scraping!