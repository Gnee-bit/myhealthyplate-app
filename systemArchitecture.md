# Eatfit - System Architecture

**Version:** 1.0  
**Last Updated:** November 2025  
**Author:** Product Team

---

## Overview

This document outlines the technical architecture for Eatfit, a mobile application that helps Nigerians make personalized healthy food choices. 

---

## 1. Technology Stack

### Frontend: Mobile Application
**Technology:** React Native


**Why I choose it:**
-  One codebase works on both iOS and Android (saves development time and cost)
-  Large community with lots of resources and libraries available

**Key features it will handle:**
- User interface and interactions
- Barcode scanning using device camera
- Offline data storage for previously scanned foods
- Displaying health recommendations

---

### Backend: Application Server
**Technology:** Node.js with Express.js


**Why I choose it:**
- Uses JavaScript (same language as our mobile app - easier for the team)
- Easy to integrate with external APIs like nutrition databases


**Key responsibilities:**
- Processing food health checks based on user profiles
- Managing user authentication and authorization
- Communicating with nutrition APIs
- Storing and retrieving data from the database
- Running the personalization logic

---

### Database: Data Storage
**Technology:** MongoDB


**Why I choose it:**
- Perfect for health profiles that vary widely between users (some have diabetes, others have allergies, etc.)
- FREE tier available (512MB storage - perfect for MVP)
- Good for apps that need to iterate and change quickly

**What we'll store:**
- User profiles (name, email, health conditions, allergies, dietary preferences)
- Food nutrition data (cached from API to reduce calls)
- Nigerian local foods database (manually curated)
- Scan history (what foods users have checked)
- Health insights and analytics data

---

### Authentication
**Technology:** Firebase Authentication


**Why I choose it:**
- Industry-standard security (used by millions of apps)
- Handles complex things like password reset, email verification
- Supports social login (Google, Apple) out of the box
- FREE tier supports up to 10,000 users (perfect for MVP)

---

### External APIs: Food & Nutrition Data

#### Primary: Open Food Facts API


**Why I choose it:**
- **Completely FREE** - no API costs (critical for intern project)
- Already includes Nigerian products (Indomie, Peak Milk, Golden Penny, etc.)
- Community-driven - users worldwide add products daily
- Barcode lookup support (essential for our scanner feature)
- Detailed nutritional information and ingredient lists


**What it covers:**
- All packaged foods with barcodes sold in Nigeria
- Local Nigerian brands already in the database
- International brands available in Nigerian markets
- User-contributed products (community keeps it updated)

**API Endpoint Example:**
```
GET https://world.openfoodfacts.org/api/v0/product/{barcode}.json
```

#### Backup: USDA FoodData Central API


**Why I include it:**
- Completely FREE
- Good for generic/unbranded foods (e.g., "banana", "rice")
- Reliable government data source
- Used when Open Food Facts doesn't have data

---

### Nigerian Local Foods Database

**What it is:** A curated collection of traditional Nigerian dishes manually added to our MongoDB database.

**Why we need it:**
- Traditional Nigerian foods don't have barcodes (jollof rice, egusi soup, pounded yam, etc.)
- External APIs don't cover local dishes adequately
- Critical for market relevance in Nigeria
- Shows understanding of local context

**How we'll build it**

**Phase 1**
- Manually research and add 20-30 most common Nigerian foods
- Use free sources:
  - Online nutrition databases
  - Nigerian food blogs and websites
  - Estimated values from similar foods
- Store as simple JSON documents in MongoDB
- Clearly mark as "estimated values"

**Starter List (Examples):**
- Rice dishes: Jollof rice, fried rice, white rice, ofada rice
- Soups: Egusi soup, ogbono soup, efo riro, edikang ikong, banga soup
- Swallows: Pounded yam, eba, fufu, amala, semovita
- Proteins: Suya, grilled fish, fried chicken, goat meat
- Snacks: Akara, moin moin, puff puff, chin chin
- Staples: Plantain (fried/boiled), beans, yam

**Data Structure Example:**
```json
{
  "name": "Jollof Rice",
  "category": "Rice Dish",
  "servingSize": "1 plate (300g)",
  "calories": 350,
  "protein": 8,
  "carbs": 65,
  "fat": 7,
  "fiber": 2,
  "sodium": 450,
  "commonAllergens": ["none"],
  "isEstimate": true,
  "notes": "Values may vary based on preparation method",
  "dateAdded": "2025-11-08"
}
```

**Phase 2 (Post-MVP):**
- Allow users to request foods to be added
- Gradually expand to 100+ foods
- Community contributions (moderated)
- Partner with nutritionists for validation (when funded)

**Why this approach works:**
- Zero cost to implement
- Demonstrates local market understanding
- Scalable foundation for future expansion
- Users will appreciate even basic coverage

---

## 2. System Architecture Diagram

Here's how all the components work together:
```
┌────────────────────────────────────────────────────┐
│                                                    │
│         USER'S MOBILE PHONE (Nigeria)              │
│                                                    │
│   ┌────────────────────────────────────────┐     │
│   │                                        │     │
│   │    Eatfit Mobile App           │     │
│   │         (React Native)                 │     │
│   │                                        │     │
│   │  • Home Screen                         │     │
│   │  • Barcode Scanner                     │     │
│   │  • Health Profile                      │     │
│   │  • Food Search                         │     │
│   │  • Recommendations                     │     │
│   │                                        │     │
│   └───────────────┬────────────────────────┘     │
│                   │                              │
└───────────────────┼──────────────────────────────┘
                    │
                    │ HTTPS / Internet
                    │ (Secure communication)
                    │
        ┌───────────▼────────────┐
        │                        │
        │   Backend API Server   │
        │      (Node.js +        │
        │      Express.js)       │
        │                        │
        │  • Authentication      │
        │  • Food Health Logic   │
        │  • User Management     │
        │  • API Integration     │
        │  • Personalization     │
        │                        │
        └──┬─────────┬──────┬────┘
           │         │      │
           │         │      │
    ───────┼─────────┼──────┼────────
           │         │      │
           │         │      │
    ┌──────▼─────┐  │  ┌───▼────────────────┐
    │            │  │  │                    │
    │  Firebase  │  │  │  Open Food Facts   │
    │    Auth    │  │  │       API          │
    │            │  │  │  (FREE - Packaged  │
    │ (Handles   │  │  │   foods/barcodes)  │
    │  login)    │  │  │                    │
    │   FREE     │  │  └────────────────────┘
    │            │  │
    └────────────┘  │
                    │
              ┌─────▼──────────────────────┐
              │                            │
              │      MongoDB Database      │
              │     (MongoDB Atlas)        │
              │                            │
              │  • User profiles & data    │
              │  • Cached food data        │
              │  • Nigerian local foods 🇳🇬│
              │  • Scan history            │
              │                            │
              │         FREE tier          │
              │                            │
              └────────────────────────────┘
```

---

## 3. How Components Communicate

### Step-by-Step: User Scans a Packaged Food Item

Let me walk through what happens when a user scans a barcode on packaged food:

**Step 1:** User opens the app and taps "Scan Barcode"
- The mobile app activates the phone's camera using react-native-camera

**Step 2:** User points camera at product barcode (e.g., Indomie noodles)
- The app reads the barcode number (e.g., "8995899470626")

**Step 3:** App sends barcode to our backend server
- Request sent over secure HTTPS connection
- Includes user's authentication token from Firebase

**Step 4:** Backend checks if we've seen this barcode before
- Searches MongoDB cache for this barcode
- If found in cache (faster, free), skip to Step 7

**Step 5:** If not cached, backend calls Open Food Facts API
- Sends: `GET https://world.openfoodfacts.org/api/v0/product/8995899470626.json`
- Receives: Detailed product information (name, brand, nutrients, ingredients)
- This is FREE - no API costs

**Step 6:** Backend saves nutrition data to MongoDB
- Caches it for future lookups (faster next time)
- Reduces API calls for popular products

**Step 7:** Backend retrieves user's health profile from MongoDB
- Gets user's allergies (e.g., peanut allergy)
- Gets health conditions (e.g., hypertension, diabetes)
- Gets dietary preferences (e.g., low-sodium diet)

**Step 8:** Backend runs personalization algorithm
- Compares food nutrients against user's health needs
- Example checks:
  - High sodium? Bad for hypertension
  - Contains peanuts? Alert for allergy
  - High sugar? Caution for diabetes
- Calculates a health score (0-100)
- Generates personalized explanation

**Step 9:** Backend sends result back to mobile app
- JSON response with health assessment:
```json
{
  "foodName": "Indomie Chicken Noodles",
  "healthScore": 45,
  "isHealthy": false,
  "reasoning": "High in sodium (900mg) - not ideal for your blood pressure condition",
  "warnings": ["High sodium content"],
  "recommendation": "Consider low-sodium alternatives"
}
```

**Step 10:** Mobile app displays results to user
- Color-coded health score:
  - 🟢 Green (70-100): Good choice
  - 🟡 Yellow (40-69): Okay in moderation
  - 🔴 Red (0-39): Avoid or limit
- Shows detailed explanation
- Saves to user's scan history in MongoDB

---

### Step-by-Step: User Searches for Nigerian Local Food

What happens when searching for traditional Nigerian food without a barcode:

**Step 1:** User types "egusi soup" in search bar

**Step 2:** App sends search query to backend

**Step 3:** Backend checks Nigerian local foods in MongoDB
- Searches our manually-curated Nigerian food collection
- Finds "Egusi Soup" entry

**Step 4:** Backend retrieves the stored nutrition data
```json
{
  "name": "Egusi Soup",
  "calories": 280,
  "protein": 12,
  "fat": 18,
  "carbs": 15,
  "isEstimate": true
}
```

**Step 5:** Backend applies personalization logic
- Same algorithm as packaged foods
- Checks against user health profile

**Step 6:** Backend returns personalized assessment

**Step 7:** App displays results with disclaimer
- Shows nutrition information
- Includes note: "Estimated values - actual content may vary by preparation"

---

### Data Flow Diagram
```
USER ACTION → Mobile App → Backend API → Data Sources
                  ↑              ↓
                  └──────────────┘
                   (Response back)

Example 1 - Barcode Scan:
Scan barcode → App → Backend → Open Food Facts API (FREE)
                            → MongoDB (user profile)
                            → Calculate health score
               ← Display results with color coding

Example 2 - Local Food Search:
Search "jollof" → App → Backend → MongoDB (Nigerian foods)
                                → MongoDB (user profile)
                                → Calculate health score
                  ← Display results with estimate note
```

---

## 4. Why This Approach is Technically Feasible

### 4.1 Proven Technology Choices

**React Native:**
- Used by Instagram (1B+ users), Facebook Marketplace, Discord
- Mature framework (released 2015, constantly updated)
- Thousands of successful apps in production
- Works perfectly in Nigeria (low bandwidth support)

**Node.js + Express:**
- Handles millions of concurrent requests
- 10+ years of production use
- Lightweight - runs on minimal infrastructure

**MongoDB:**
- Trusted for healthcare applications
- Proven scalability (can handle billions of records)
- Free tier is generous for MVPs

**Firebase Auth:**
- Owned by Google, used by millions of apps
- 99.95% uptime guarantee
- Automatically handles security patches
- Works reliably in African markets

**Open Food Facts:**
- Powers apps like Yuka (10M+ downloads)
- Community of 30,000+ contributors worldwide
- Nigerian products already in database
- Proven API reliability

---

### 4.2 Manageable Complexity

**Clear Separation of Concerns:**
- Mobile app focuses on user interface
- Backend handles business logic
- Database just stores data
- Each part can be built and tested independently

**Well-Documented:**
- All chosen technologies have extensive documentation
- Large communities mean help is readily available
- Thousands of tutorials and YouTube videos
- Stack Overflow has millions of answered questions

**No Custom Infrastructure:**
- MongoDB Atlas (cloud-hosted, no server management)
- Firebase (fully managed by Google)
- Open Food Facts (external service, they handle uptime)
- **We focus on building features, not managing servers**


---

### 4.3 Nigerian Market Considerations

**Challenge:** International nutrition APIs don't cover Nigerian local foods comprehensively.

**Our Solution - Hybrid Approach:**

1. **Packaged/Imported Foods (with barcodes):**
   - Use Open Food Facts API (FREE)
   - Already has Peak Milk, Indomie, Golden Penny, etc.
   - Community keeps it updated

2. **Traditional Nigerian Foods:**
   - Manual database (20-30 foods for MVP)
   - Research using free online sources
   - Clearly marked as estimates
   - Expandable based on user feedback

**Why this works for Nigeria:**
- Most packaged foods in supermarkets have barcodes → covered by Open Food Facts
- Traditional foods are finite list (top 100 covers most daily meals)
- Nigerians are tech-savvy and will appreciate even basic coverage
- Can crowdsource additional foods post-launch

**Data Sources for Nigerian Foods:**
- Google searches for nutritional content
- Nigerian food blogs (Nigerian Food TV, Sisi Yemmie, etc.)
- Comparison with similar foods in databases
- Conservative estimates where exact data unavailable

---



### 4.4 Budget & Resource Considerations

**Infrastructure Costs (Monthly):**

| Service | Free Tier | When We'll Pay | Estimated Cost |
|---------|-----------|----------------|----------------|
| MongoDB Atlas | 512MB storage | After ~5,000 users | ₦20,000-₦50,000/month |
| Firebase Auth | 10,000 users | After 10,000 users | ₦30,000-₦60,000/month |
| Open Food Facts API | Unlimited | Never (it's free!) | ₦0 |
| Backend Hosting | Varies | From day 1 (minimal) | ₦10,000-₦30,000/month |

**Total Starting Cost: ₦0-₦10,000/month for MVP**
**Total at 1,000 users: ₦10,000-₦30,000/month**



**Why this is feasible:**
- No upfront infrastructure investment
- All critical services have generous free tiers
- No expensive API subscriptions
- Manual work replaces expensive partnerships
- Can launch and test market fit before spending money
- Scalable - costs only increase as users grow

---

### 4.5 Risk Management

| Risk | Likelihood | Impact | Mitigation Strategy |
|------|------------|--------|-------------------|
| **Open Food Facts API downtime** | Low | Medium | Cache all lookups in MongoDB; show cached data if API fails |
| **Limited Nigerian food coverage** | High | Medium | Start with top 30 foods; add more based on user requests; set expectations clearly |
| **Barcode scanning doesn't work well** | Medium | High | Use proven library (react-native-camera); test extensively; provide manual entry option |
| **Algorithm not accurate enough** | Medium | Medium | Start simple with rule-based logic; improve iteratively with user feedback |
| **Team doesn't know React Native** | Medium | Low | Extensive learning resources available; similar to React web; 2-week learning curve |
| **Free tiers run out** | Low | Medium | Monitor usage closely; optimize queries; seek funding before hitting limits |
| **Users expect more foods in database** | High | Low | Set expectations upfront; show "Request Food" option; add popular requests quickly |



---

## 5. Technical Feasibility Summary

### Is This Technically Possible?  **Yes, Absolutely.**

**Evidence:**

1. **Similar apps exist and succeed:**
   - MyFitnessPal (250M+ users) - similar tech stack
   - Yuka (25M+ users) - uses Open Food Facts API
   - Fooducate (10M+ users) - barcode scanning works
   - All prove this approach is viable

2. **All technologies are proven at scale:**
   - React Native powers billion-user apps
   - Node.js handles Netflix-level traffic
   - MongoDB trusted by enterprises
   - Firebase used by millions of apps

3. **No custom AI/ML required for MVP:**
   - Simple rule-based logic ("high sodium = bad for hypertension")
   - Can improve sophistication over time
   - Don't need complex algorithms to provide value

4. **Free external APIs handle the hard parts:**
   - Open Food Facts has the food database
   - We don't need to build/maintain millions of products
   - Just focus on personalization logic

5. **Manual work is manageable:**
   - 20-30 Nigerian foods = 1-2 weeks of research
   - Clear, achievable scope
   - Can expand incrementally

### Biggest Technical Challenges:

**Challenge 1: Barcode Scanning Accuracy**
- **Solution:** Use battle-tested library (react-native-camera)
- **Backup:** Manual barcode entry option
- **Risk Level:** Low (solved problem)

**Challenge 2: Nigerian Food Data Accuracy**
- **Solution:** Start with estimates, mark clearly, improve over time
- **Backup:** Users can report issues
- **Risk Level:** Medium (but manageable)

**Challenge 3: Personalization Algorithm**
- **Solution:** Start simple (if sodium > 500mg AND user has hypertension → score down)
- **Backup:** Can refine based on user feedback
- **Risk Level:** Low (can iterate)

---


## Conclusion

Eatfit's architecture is straightforward, uses battle-tested free technologies, and is absolutely feasible for an intern team to build. The hybrid approach (free API + manual Nigerian foods) balances ambition with practicality.

**Key Success Factors:**
- Zero infrastructure costs to start
- Realistic scope for intern timeline
- Proven technologies reduce risk
- Nigerian market context addressed practically
- Clear path from MVP to scale
- Iterative approach allows learning and adaptation


**This is not just technically possible - it's the smart way to validate the concept before major investment.**

---

*Note: As a Product Manager, I understand these high-level architectural decisions and their trade-offs. Detailed implementation (specific code, database queries, security protocols, optimization techniques) would be developed in collaboration with the engineering team during actual development. This document serves as a foundation for technical discussions and demonstrates the product's feasibility.*

---

**Document Version History:**
- v1.0 (Nov 2025) - Initial architecture with Nigerian market considerations and free-tier optimization