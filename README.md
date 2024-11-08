const express = require('express');
const bodyParser = require('body-parser');
const axios = require('axios');
const cheerio = require('cheerio');

const app = express();
const port = 3000;

app.use(bodyParser.json());

app.post('/scrape', async (req, res) => {
    const websites = req.body.websites;
    const latestPosts = [];

    for (let website of websites) {
        try {
            const response = await axios.get(website);
            const $ = cheerio.load(response.data);
            
            // Try to find the latest post based on typical HTML structures
            const latestPostUrl = $('article a').first().attr('href') || $('a').first().attr('href');
            const postUrl = latestPostUrl ? new URL(latestPostUrl, website).href : website;

            latestPosts.push({ website, url: postUrl });
        } catch (error) {
            latestPosts.push({ website, url: website }); // If error, return the website URL
        }
    }

    res.json({ latestPosts });
});

app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});
