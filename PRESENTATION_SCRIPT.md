# BrandBondhu - Presentation Script (Group 2)

## 🎬 1. Introduction
**Visual Action (Web View):** Show the BrandBondhu Landing Page / Login Screen.

**Script:**
> "AssalamuAlaikum sir, I am tanjima mahmud and I will be covering my features of our 470 project

---

## 🔐 2. Auth, Roles, and Team Access
**Visual Action (Web View):** Log in as a 'Seller'. Then navigate to the Team/Collaborator invite section in the dashboard.

**Script:**
> "First is Authentication and Role-Based Control. where a person can login as Sellers, Designers, or Collaborators. On the dashboard, a Seller can invite a team member and assign them specific permissions."

**Code Highlight:**
* **Open:** `backend/controllers/profileController.js` (or `backend/middleware/authMiddleware.js`)
* **Show:** Highlight the `inviteCollaborator` logic.
* **Script:** "If we look at the backend, we use JWT for secure sessions. In our profile controller, when inviting a collaborator, we create a new user tied to the parent's ID and push their specific permissions into an array. Our React frontend then reads this and dynamically hides UI tabs based on these permissions."

---

## 📢 3. AI Content Publishing & Scheduling
**Visual Action (Web View):** Go to the "Content Manager" page. Input some product details, generate a caption using the AI tool, and schedule a post.

**Script:**
> " Next is the AI Content Publisher. A seller can input basic product details, and the system will generate appropriate captions. I also built a scheduling function, that uses a mock publishing system to simulates the API interaction."

**Code Highlight:**
* **Open:** `backend/controllers/contentController.js`
* **Show:** Highlight the `CAPTION_GENERATOR` block and then scroll down to highlight `triggerPostReminders`.
* **Script:** "Here is our caption generator logic. But more importantly, this `triggerPostReminders` function acts as our Cron job. It checks the database for scheduled posts. If a live Facebook token exists, it pushes to the Graph API. If not, it safely mocks the success and updates the database status to 'Published', proving our logic works seamlessly either way."

---

## 🤖 4. Automated Customer FAQ Assistant
**Visual Action (Web View):** Show the "FAQ Manager" page. Add a new FAQ with some keywords. (If you have a UI way to trigger a mock comment, do it, otherwise just explain the flow while looking at the manager).

**Script:**
> "Similarly, there is Auto-Reply Assistant where Sellers can define FAQs and trigger keywords. When a customer comments on their page, the system will intercept it and reply automatically."

**Code Highlight:**
* **Open:** `backend/controllers/faqController.js`
* **Show:** Highlight the `processCommentForFAQ` function.
* **Script:** "In our `processCommentForFAQ` function, you can see the 'brain' of the auto-responder. We take the incoming comment, convert it to lowercase, and check if it matches any saved keywords using Javascript array methods. If a match is found, we trigger our Facebook Graph API utility to post the reply."

---

## 🎨 5. Freelance Designer Marketplace
**Visual Action (Web View):** Show the marketplace or a specific "Design Job" view, preferably one that is currently active.

**Script:**
> "BrandBondhu also connects sellers with freelance designers. We had to manage complex states like escrow funds and job approvals. We implemented a robust state machine in our database to handle this."

**Code Highlight:**
* **Open:** `backend/models/DesignJob.js`
* **Show:** Highlight the `status` and `escrowStatus` enums in the model schema.
* **Script:** "In our Mongoose model, the job status strictly moves from 'Requested' to 'Delivered' to 'Approved'. We also track 'escrowStatus'. Our controllers enforce these transitions, ensuring that funds are only 'Released' or 'Refunded' under the correct conditions, providing a secure simulated environment."

---

## 💳 6. Subscriptions, Payments & AI Credits
**Visual Action (Web View):** Go to the "Billing" or "Upgrade Plan" page. Click through the mock bKash/payment gateway interface.

**Script:**
> "To monetize the platform, we implemented a tiered subscription system with AI credit metering. Every time a user generates AI content, it deducts a credit. We also built a mock payment gateway that simulates the bKash or SSLCommerz checkout experience."

**Code Highlight:**
* **Open:** `backend/controllers/paymentController.js`
* **Show:** Highlight `initiatePayment` and then `confirmPayment`.
* **Script:** "Here, `initiatePayment` creates a 'Pending' transaction. Once the user completes the mock UI flow, `confirmPayment` is called. It finds the transaction, marks it 'Completed', upgrades the user's subscription tier, and refills their AI credits."

---

## 📊 7. Analytics & Competitor Tracking
**Visual Action (Web View):** Show the Analytics Dashboard with the charts rendered.

**Script:**
> "Finally, we have our Analytics dashboard. It performs sentiment analysis on comments and tracks competitor metrics. Since web scraping social media is heavily blocked, we built a data simulation module for demonstration."

**Code Highlight:**
* **Open:** `backend/controllers/analyticsController.js`
* **Show:** Highlight `getSentimentAnalysis` or `addCompetitor`.
* **Script:** "In `getSentimentAnalysis`, we filter and count 'positive', 'negative', and 'neutral' comments to feed our React charts. For competitors, we simulate data generation here, ensuring our frontend has realistic metrics to display and test against."

---

## 👋 8. Conclusion
**Visual Action (Web View):** Return to the main Dashboard or a 'Thank You' slide/screen.

**Script:**
> "To summarize, Group 2 successfully built the entire operational core of BrandBondhu. We handled secure authentication, complex state management for design jobs, dynamic AI credit metering, and robust simulated integrations for payments and social media. Thank you, we are now open to questions."
