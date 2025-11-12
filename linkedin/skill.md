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

**When:** User provides a person's name but NOT a LinkedIn URL. Apify scrapers need LinkedIn URL as input. 

**Action:** Use API Call to Google Programmatic Search Engine

https://www.googleapis.com/customsearch/v1?q={{encodeURL(1.Search_query)}}&cx=e749a6c5c29a5470f&key=AIzaSyDjWZ7YmALNnkFpS9cFJzlKvePecLouZfg&num=10&gl=cz

**Best practices:**
- Include employer name if known (increases accuracy)
- Include job title if ambiguous name
- Example: "Satya Nadella Microsoft" is better than just "Satya Nadella"

**Output:** You will receive the list of search result. 
- Write a code to clean it from all coding characters to let AI process only the clean text
- Process the search result by AI and select the most likely LinkedIn profile URL corresponding to the search query. 

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
