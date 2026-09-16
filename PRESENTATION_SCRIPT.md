# BrandBondhu - Presentation Script (Group 2)

## 🎬 0. Introduction
**Visual Action (Web View):** Show the BrandBondhu Landing Page / Login Screen.

**Script:**
> "AssalamuAlaikum sir, I am tanjima mahmud and I will be covering my features of our 470 project

---

## 📢 1. Multi platform Content Publishing & Scheduling
**Visual Action (Web View):** Go to the "Content Manager" page. Input some product details, generate a caption using the AI tool, and schedule a post.

**Script:**
> " First is the Multi platform Content Publisher. A seller can input basic product details, and the system will generate appropriate captions. I also built a scheduling function, that uses a mock publishing system to simulates the API interaction."

**Code Highlight:**
* **Open:** `backend/controllers/contentController.js`
* **Show:** Highlight the `CAPTION_GENERATOR` block and then scroll down to highlight `triggerPostReminders`.

* **Script:** 
> "Here is the caption generator logic. But more importantly, this `triggerPostReminders` function acts as the Cron job. It checks the database for scheduled posts. If a live Facebook token exists, it pushes to the Graph API. If not, it safely mimics the success and updates the database status to 'Published'. "

---

## 🤖 2. Automated Customer FAQ Assistant
**Visual Action (Web View):** Show the "FAQ Manager" page. Add a new FAQ with some keywords. (If you have a UI way to trigger a mock comment, do it, otherwise just explain the flow while looking at the manager).

**Script:**
> "Secondly, I built an Auto-Reply Assistant. Sellers can define FAQs and trigger keywords. When a customer comments on their page, my system intercepts it and replies automatically."

**Code Highlight:**
* **Open:** `backend/controllers/faqController.js`
* **Show:** Highlight the `processCommentForFAQ` function.
* **Script:** 
> "In my `processCommentForFAQ` function, you can see the 'brain' of the auto-responder. I take the incoming comment, convert it to lowercase, and check if it matches any saved keywords using Javascript array methods. If a match is found, I trigger my Facebook Graph API utility to post the reply."

---

## 📈 3. Facebook Comment Sentiment Analysis
**Visual Action (Web View):** Show the Analytics Dashboard focusing on the Sentiment Analysis charts.

**Script:**
> "For analytics, I implemented a Sentiment Analysis feature. It analyzes the incoming Facebook comments to give sellers an immediate read on customer satisfaction."

**Code Highlight:**
* **Open:** `backend/controllers/analyticsController.js`
* **Show:** Highlight the `getSentimentAnalysis` function.
* **Script:** 
> "In `getSentimentAnalysis`, I query the social comments database. I then filter and calculate the distribution of 'positive', 'negative', and 'neutral' sentiments. This processed data is what feeds directly into the React charts I set up on the frontend."

---

## 🕵️ 5. Competitor Tracking
**Visual Action (Web View):** Show the Competitor Tracking section in the Dashboard.

**Script:**
> "Another tool I built for sellers is Competitor Tracking. Users can input a competitor's page URL, and the system begins tracking their metrics. Since scraping social media is heavily blocked without enterprise API access, I built a simulation module to demonstrate how the data would be handled."

**Code Highlight:**
* **Open:** `backend/controllers/analyticsController.js`
* **Show:** Highlight the `addCompetitor` function.

* **Script:** 
> "In the `addCompetitor` function, I simulate the data retrieval process, generating realistic metrics like follower counts and engagement rates. This ensures the database schema and frontend tracking logic are fully operational and ready to swap in a live data provider."

---

## 📊 6. Analytics Dashboard (Conclusion)
**Visual Action (Web View):** Zoom out to show the full Analytics Dashboard with all metrics rendered together.

**Script:**
> "To tie it all together, I built this centralized Analytics Dashboard. It aggregates the sentiment data, competitor metrics, and post performance into one cohesive view. By handling the complete data flow—from backend processing controllers to the React frontend UI, I ensured a seamless experience. Thank you."


