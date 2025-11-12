---
name: linkedin-intelligence
description: LinkedIn data extraction workflow for profile research, posts analysis, and interaction tracking. Handles LinkedIn profile URL discovery, detailed profile scraping, post extraction, and engagement analysis. Gives the list of tools and Apify actors to use.
---

# LinkedIn Intelligence Skill

## When to Use This Skill
- User asks to "research someone on LinkedIn"
- User needs candidate profile details
- User wants to analyze someone's LinkedIn activity
- User mentions "LinkedIn posts" or "LinkedIn engagement"
- User provides a person's name for professional research

---

## PROCESS 1: Find LinkedIn Profile URL

Trigger:
User provides a person's name but does not provide a LinkedIn profile URL.

### Workflow:

Build the search query:

Combine all information provided by the user (name, employer, location, etc.) into the search string.

Example: "Satya Nadella Microsoft" is better than just "Satya Nadella".

Encode the search query for safe URL usage:

Use a function like encodeURIComponent() to ensure all special characters (such as "ž") are properly encoded for use in the URL.

Make an HTTP GET request to the Google Programmable Search Engine API:
Endpoint:
https://www.googleapis.com/customsearch/v1
Required parameters:

q (the encoded search query)
cx (Custom Search Engine ID)
key ( )
num (number of results, e.g., 10)
gl (country code, e.g., "cz")

Sample code snippet:

python
import requests
from urllib.parse import quote

search_query = "Jan Žák Employer Prague"
encoded_query = quote(search_query)  # This encodes "ž" and other special characters

params = {
  "q": encoded_query,
  "cx": "e749a6c5c29a5470f",
  "key": "AIzaSyDjWZ7YmALNnkFpS9cFJzlKvePec",
  "num": 10,
  "gl": "cz"
}
response = requests.get("https://www.googleapis.com/customsearch/v1", params=params)
search_results = response.json()
Clean the API results:

Extract only the text relevant for LinkedIn profiles.

Remove any HTML or coding artifacts.

Focus on links from the linkedin.com/in/ domain.

### Select the best LinkedIn profile URL:

Use AI to evaluate the search results and select the most relevant LinkedIn profile URL based on name, employer, and description.

If no suitable profile is found, return "not found".

### Output:

The LinkedIn profile URL (e.g., https://www.linkedin.com/in/jan-zak)

If not found: "not found"

Notes & Best Practices:

Always use proper URL encoding when building the API query (to handle characters like "ž", "ě", "ř", etc.).

Never guess or generate a LinkedIn URL; always get it from search results.

Return only valid LinkedIn profile URLs or a clear message if no result is found.

---

## PROCESS 2: Extract Complete Profile Details

**When:** The user wants to know career related information, skills, professional background of an individual. 
From the context of user request it is clear the data from LinkedIn will be helpful.

**Action:** Scrape comprehensive profile information

**Apify Actor to use:** 

apimaestro/linkedin-profile-batch-scraper-no-cookies-required

Based on the context, provide either full profile information or only parts related to the specific request.

---

## PROCESS 3: Extract Recent Posts

**When:** You need to analyze what the person posts about

**Action:** Retrieve the person's recent LinkedIn posts

**Apify Actor to use:**
```
harvestapi/linkedin-profile-posts
```

Provide full details below unless you get different orders.
- Post content (text)
- Post media (images, videos, documents)
- Engagement metrics (likes, comments, shares)
- Post date and time
- Post type (original post, repost, article)

**Configuration:**
- Default: Last 10 posts
- Include quote posts: Yes

**Input:** LinkedIn profile URL from Step 1

----


## Error Handling

**Profile not found:**
- Verify name spelling with user
- Ask for additional context (company, location)
- Try alternative search queries

**Rate limits:**
- If Apify actor hits limits, inform user
- Suggest waiting or upgrading Apify plan

---

## Output Format & Behavior Instructions

Act like a LinkedIn specialist, social media behaviour expert.
You have great knowledge of Linkedin, profiles, how to read between the lines, infer behaviour and motivations.
You are able to make deep psyhological anylsis basedon people's digital footprin.t 
