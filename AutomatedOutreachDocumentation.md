# Automated Outreach Strategy Workstream
## Final Project Documentation
   
**Project:** Jamtown Artist Outreach Automation Initiative  
**Workstream:** Automated Outreach Strategy (AOS)  
**Duration:** September 30, 2024 – December 1, 2024  
**Status:** **Complete**

---

## Executive Summary

The **Automated Outreach Strategy** workstream was designed to automate Jamtown's artist acquisition process, enabling efficient outreach to potential artist partners without requiring manual effort at scale. The core objective was to develop a systematic, scalable approach to discover, identify, and contact emerging and mid-tier artists across the United States, thereby expanding Jamtown's artist community and increasing brand awareness among the target demographic.

This workstream serves a critical function within the overall Jamtown project ecosystem: it acts as the **acquisition engine** that sources new artist partners and generates the foundational data required for downstream AI matching algorithms. By automating the artist discovery and initial outreach pipeline, this workstream directly enables Jamtown to populate its database with artist profiles (including bio, interests, causes, and performance preferences), which the AI workstream then leverages to match artists with nonprofit event opportunities. Ultimately, the AOS workstream is instrumental in achieving Jamtown's mission of connecting independent and emerging artists with meaningful performance stages while simultaneously building a rich, validated artist dataset that powers the platform's intelligent matching capabilities.

---

## 1. Objectives & Success Criteria

### Primary Objectives

1. **Develop an automated artist discovery pipeline** that identifies and extracts Spotify artist profile URLs at scale without reliance on costly official APIs.
2. **Establish a data enrichment process** that retrieves critical artist information (Instagram profiles, follower counts, monthly listeners) necessary for audience segmentation and AI matching.
3. **Implement automated outreach infrastructure** that enables direct, personalized communication with target artists via Instagram Direct Messages.
4. **Generate a validated, segmented artist database** that feeds directly into the Jamtown platform and AI matching engine, with artists pre-qualified by geography and audience size.

### Key Performance Indicators (KPIs)

The success of this workstream will be measured by the following metrics:

- **Response Rate (%)**: Percentage of artists who respond to initial outreach messages.
- **Sign-Up Conversion Rate (%)**: Percentage of respondents who complete the artist profile signup and join the Jamtown talent community.
- **Database Entries Per Week**: Number of new, validated artist profiles added to the Jamtown database per week.
- **Data Quality Score**: Percentage of artist records with complete information (bio, causes, contact info, social links) ready for AI processing.
- **Geographic Coverage**: Number of qualified artists by state/region to ensure national representation.

---

## 2. Process & Methodology

The Automated Outreach Strategy was developed across five distinct phases, each with specific deliverables and technical implementations.

### Phase 1: Scope & Outline of Workstreams (Week 1 | Sept 30 – Oct 6)

**Objective:** Define the AOS workstream's role within the broader Jamtown project and establish technical and operational boundaries.

**Activities:**
- Stakeholder alignment meetings to clarify AOS dependencies with AI and SEO workstreams.
- Defined data requirements: artist profiles needed by AI workstream (bio, causes, performance preferences, social links).
- Identified bottlenecks in manual outreach and quantified the need for automation.
- Established success criteria and KPI framework.

**Key Outputs:**
- Workstream charter and objective document.
- Data schema specification for artist profiles.
- Integration requirements for the Framer website database and AI platform.

---

### Phase 2: Research – Ways to Obtain Artist Information (Week 2 | Oct 7 – Oct 13)

**Objective:** Evaluate all viable approaches to artist data collection and select the most scalable, ethical, and cost-effective solution.

**Activities:**

#### Investigation: Spotify API
- Evaluated the official Spotify Web API for artist metadata retrieval.
- **Finding:** Spotify API provides limited artist information (genres, follower count, monthly listeners) but lacks contact information, Instagram profiles, and bio data necessary for AI matching and direct outreach.
- **Conclusion:** Official API alone insufficient; alternative data sources required.

#### Investigation: Web Scraping Approaches
- Researched direct Spotify website scraping using Selenium and Playwright.
- **Finding:** While technically feasible, large-scale web scraping of Spotify violates their Terms of Service and raises ethical and legal concerns regarding data ownership and artist privacy.
- **Conclusion:** Ethical concerns and ToS violations ruled out as primary approach.

#### Investigation: Third-Party Data Aggregators
- Evaluated commercial music industry databases and artist platforms.
- **Finding:** Cost-prohibitive for startup; limited coverage of emerging/independent artists; inflexible data structures.

#### Hybrid Solution: Artist URL Scraping + Instagram Enrichment
- **Proposed Approach:** Scrape Spotify artist profile URLs (which are public, static, and not rate-limited) using headless browser automation, then use a dedicated Instagram scraper (Apify) to extract contact and audience data from artist Instagram profiles.
- **Rationale:** Artist profile URLs are public metadata; Instagram profiles contain the contact, bio, and audience information needed for outreach and AI matching. This approach respects Spotify's ToS while ethically sourcing public artist information.
- **Conclusion:** Selected as primary approach.

**Key Outputs:**
- Comparative analysis report of data collection approaches.
- Technical feasibility assessment: Playwright-based URL scraper + Apify Instagram enrichment.
- Risk mitigation strategy for ethical data collection and artist privacy.

---

### Phase 3: Development – Algorithmic Artist Information Pipeline (Weeks 3–4 | Oct 14 – Oct 27)

**Objective:** Build and validate the core technical infrastructure for discovering and enriching artist data at scale.

#### 3A. Spotify Artist URL Discovery Algorithm

**Technology Stack:**
- Language: Python 3.10+
- Core Library: Playwright (headless Chromium automation)
- Supporting Libraries: Pandas (data manipulation), standard file I/O

**Architecture & Approach:**

The `discover_artists.py` script implements a distributed, resumable web scraper that discovers Spotify artist profile URLs without using the official API:

1. **Query Planning:** Generates a diverse search query space (artist names, genre terms, trending keywords) to maximize coverage of the artist ecosystem.
2. **Distributed Crawling:** Spawns multiple Playwright browser contexts (default: 6 workers) that run in parallel, each independently querying Spotify search results.
3. **URL Extraction & Deduplication:** Each worker scrolls through search result pages, extracting artist profile URLs and persisting them to a shared `urls.txt` file with automatic deduplication.
4. **Resumable Execution:** Tracks progress between runs; can resume from a prior checkpoint or start fresh with the `--fresh` flag.
5. **Resilience:** Automatically handles transient network errors and Spotify rate-limiting; logs per-worker progress and warnings.

**Key Technical Parameters:**

| Parameter | Description | Default |
| --- | --- | --- |
| `--workers` | Number of parallel Playwright contexts | 6 |
| `--count` | Target number of unique artist URLs to discover | 1,000 |
| `--per-query-target` | Stop scrolling once this many new artists found per search term | Tuned per environment |
| `--max-scrolls` | Maximum number of result pages to scroll per query | Tuned per environment |
| `--scroll-pause-ms` | Pause duration between scroll actions (ms) | Tuned per environment |
| `--fresh` | Ignore existing `urls.txt` and start fresh | False (resume by default) |

**Execution:**
```bash
python discover_artists.py --count 100000 --fresh
```

**Outcome (Jamtown Project):**
- Discovered and validated **100,000 artist profile URLs** from Spotify.
- Stored in `urls.txt` for downstream processing.

---

#### 3B. Data Filtering & Segmentation

**Objective:** Prepare the artist URL dataset for Instagram enrichment by applying geographic and audience-size filters.

**Process:**

1. **Data Loading:** Read the full `urls.txt` into a Pandas DataFrame.
2. **Geographic Filtering:** Cross-reference artist profile metadata (Spotify API for country/region) to retain only **United States–based artists**.
3. **Audience Segmentation:** Apply sorting and filtering algorithms to identify emerging/mid-tier artists:
   - **Lower Bound:** Minimum 500 Spotify followers, 5,000 monthly listeners (filters out inactive/spam profiles).
   - **Upper Bound:** Maximum 100,000 Spotify followers, 1,000,000 monthly listeners (excludes major label artists with established PR teams; targets emerging independent artists).
4. **Validation:** Detect and flag duplicate or invalid URLs; remove corrupted records.

**Outcome:**
- Refined dataset of ~50,000–80,000 US-based, emerging/mid-tier artists ready for Instagram enrichment.

**Rationale for Audience Thresholds:**
- **Lower Bounds:** Artists below these thresholds often lack professional presence or active social engagement; outreach is unlikely to succeed.
- **Upper Bounds:** Artists above these thresholds typically have management teams, formal booking processes, and professional outreach channels; direct DM outreach is less effective and may be perceived as unprofessional.

---

### Phase 4: Testing – Apify Instagram Enrichment Pilot (Week 5 | Oct 28 – Nov 3)

**Objective:** Validate the Instagram data enrichment process and refine targeting criteria before full-scale deployment.

**Setup:**

Two **Apify packages** were selected as the Instagram data source:

1. **Spotify Artist Link Scraper** ($15/month base + per-run cost): Automatically retrieves artist Instagram links from Spotify artist pages.
   - **Initial Decision:** Purchase deemed cost-effective as a service wrapper over Spotify API data.
   
2. **Apify Instagram Artist Scraper** (Separate package; per-run cost ~$0.007/link): Extracts detailed Instagram profile metadata (follower count, bio, recent post activity, contact info).

> **Note:** The Apify account used is linked to **PJ’s Gmail account**, and all billing and usage history are tied to that login.

**Pilot Execution:**

**Round 1 – Build Phase Optimization:**
- **Initial Assessment:** Evaluated the $15/month Spotify API package for cost-benefit.
- **Finding:** Determined that a **custom in-house algorithm** could replicate the same functionality (extracting Spotify artist URLs and links) at zero recurring cost.
- **Decision:** Developed internal algorithm; cancelled the $15/month subscription.

**Round 2 – Instagram Data Enrichment Pilot:**
- **Test Cohort:** Selected first 5,000 artist URLs from the filtered dataset.
- **Apify Run:** Executed Instagram scraper on the 5,000 URLs; cost ~$35 (5,000 links × ~$0.007/link).
- **Data Returned:** Received Instagram profile data for ~1,500 artists (30% yield; reflects inactive/deleted profiles or missing Instagram links).
- **Segmentation:** Applied final filtering criteria:
  - **Geographic:** Artists with US-based Instagram location data.
  - **Audience:** 500–100,000 Instagram followers AND 5,000–1,000,000 Spotify monthly listeners.
  - **Result:** **330 qualified artists** met all targeting criteria.

**Key Insight:**
The 30% yield rate (1,500 Instagram profiles from 5,000 URLs) is attributable to:
- Artists without public Instagram profiles linked to Spotify.
- Inactive or deleted Instagram accounts.
- Incomplete or missing profile metadata.

This validated the necessity of the Apify enrichment step; direct Spotify-only data was insufficient for outreach.

**Deliverables:**
- Pilot results CSV: 330 qualified artists with Instagram handles, follower counts, and Spotify data.
- Apify configuration and rate-limit documentation.
- Cost analysis and ROI projection for full-scale runs.

---

### Phase 5: Outreach – Automated Instagram DM Campaign (Weeks 6–7+ | Nov 4 – Present)

**Objective:** Deploy automated, personalized Instagram Direct Messages to qualified artists and track engagement toward sign-ups.

#### 5A. Outreach Tool & Infrastructure

**Tool Selected:** Google Chrome Extension – **Automated Instagram DM Bot**

**Specifications:**
- **Functionality:** Accepts a message template and a list of Instagram usernames; automatically sends DMs to each user in sequence.
- **Rate Limiting:** Enforces 5–7 minute delays between each DM to avoid Instagram detection and account suspension.
- **Daily Capacity:** Limited by Chrome extension free tier; capable of sending ~210 DMs per day (free plan limitation).
- **Pro Plan Alternative:** Upgrade available but not deployed due to budget constraints; would increase daily capacity to ~500–1,000 DMs/day.

**Rationale for Instagram Over Email:**
- **Higher Engagement:** Instagram DMs generate significantly higher open and response rates than email among younger, independent artists.
- **Data Limitations:** Email addresses are difficult to extract from Spotify/Instagram profiles and are often associated with PR teams rather than individual artists.
- **Direct Access:** Instagram DMs provide direct, personal communication channel without intermediary gatekeeping.

#### 5B. Message Template & Campaign Strategy

**Message Template:**
```
Hi <Username>! I'm reaching out from Jamtown, where we connect artists
with nonprofits hosting events and performances. We can help you get matched
with events that fit your genre and schedule, grow your reach by performing
for new audiences, and handle all the logistics! Moreover, we will provide
you with a stage to perform at for causes that you care about. It's
completely free, we'd love it if you could check it out and join our
artist community here: https://traditional-reason-961104.framer.app/artist-contact
```

**Message Design Rationale:**
- **Personalization:** Addresses artist by first name to increase perceived legitimacy and engagement.
- **Value Proposition:** Clearly articulates Jamtown's core offering (free performance opportunities, audience growth, cause alignment).
- **Call-to-Action:** Direct link to artist signup form minimizes friction between interest and conversion.
- **Tone:** Professional yet approachable; positions Jamtown as a peer platform, not a commercial booking agent.

#### 5C. Campaign Execution & Constraints

**Deployment Plan:**
- **Cohort 1 (Weeks 6–7):** Outreach to first batch of 330 qualified artists from pilot test.
- **Weekly Capacity:** ~210 DMs/day × 7 days = ~1,470 DMs/week at current limitations.
- **Current Progress:** 40 DMs sent (Week 6 partial deployment; limited by free-tier DM send rate).

**Scaling Constraints:**
- **Free-Tier Limitation:** Current Chrome extension free plan limits to ~210 DMs/day. Reaching all 330 pilots + future cohorts will require:
  - **Option A:** Upgrade to Pro plan (increases capacity but adds recurring cost).
  - **Option B:** Parallel deployment across multiple accounts (manual workaround; increases operational overhead).
  - **Option C:** Longer campaign duration (accept slower timeline to work within free-tier limits).

**Trust & Legitimacy Challenges:**
- **Challenge Identified:** Artists may perceive cold Instagram DMs as spam or scams, particularly given the proliferation of predatory booking scams in the music industry.
- **Mitigation Strategies:**
  - Transparent value proposition (free platform, nonprofit focus, no upfront fees).
  - Professional message tone and branding.
  - Framer website with polished design and clear artist testimonials (once available).
  - LinkedIn/social proof linking from Jamtown official accounts.
  - Follow-up email to respondents with additional verification and team information.

**Planned Monitoring:**
- Track response rate % on first 50 outreach messages.
- Adjust message template if response rate falls below 2%.
- Document artist feedback and objections for future optimization.

---

## 3. Technical Architecture & Deliverables

### 3.1 SpotifyScraper Algorithm – Repository Overview

**GitHub Repository:** [Handover to Client]

**Project Structure:**

```
spotify-scraper/
├── discover_artists.py          # Main scraper orchestrator
├── analyze_artists.py            # Metadata analyzer & CSV reporter
├── urls.txt                      # Rolling artist URL list
├── artist_popularity.csv         # Latest analyzed/filtered artists
├── JSON Files/                   # Apify export staging area
├── requirements.txt              # Python dependencies
├── README.md                     # Full technical documentation
└── .gitignore                    # Git ignore rules
```

**Core Scripts:**

#### `discover_artists.py`
- **Purpose:** Discovers and extracts Spotify artist profile URLs at scale using Playwright-driven Chromium workers.
- **Inputs:** CLI flags (--count, --workers, --fresh, etc.)
- **Outputs:** `urls.txt` (deduplicated artist URLs)
- **Key Features:**
  - Distributed, parallel crawling with configurable worker count.
  - Resumable execution; tracks progress between runs.
  - Automatic retry logic for transient network failures.
  - Extensive logging for debugging and performance monitoring.

**Example Usage:**
```bash
python discover_artists.py --count 100000 --workers 8 --fresh
```

#### `analyze_artists.py`
- **Purpose:** Ingests raw Apify JSON exports and produces filtered, ranked artist contact lists and popularity CSVs.
- **Inputs:** Single JSON file or directory of JSON files (from Apify exports).
- **Outputs:**
  - Console output: Filtered artist list with Instagram/email contacts and audience stats.
  - `artist_popularity.csv`: Ranked, normalized popularity scores for all qualifying artists.
- **Key Features:**
  - Configurable audience thresholds (followers, monthly listeners).
  - Geographic filtering (US-based artists).
  - Contact extraction (Instagram handles, email, if available).
  - Popularity scoring algorithm.

**Example Usage:**
```bash
# Analyze single export
python analyze_artists.py spotify_artist_info_1-500.json

# Aggregate all JSON files in directory
python analyze_artists.py "JSON Files"
```

### 3.2 Data Outputs & Handover Files

**Files Delivered to Client:**

1. **`urls.txt`** – Master list of 100,000 discovered Spotify artist profile URLs.
   - Format: One URL per line.
   - Use Case: Input for future Apify runs or artist discovery iterations.

2. **`artist_popularity.csv`** – Filtered and ranked artist dataset.
   - Columns: Artist Name, Spotify Profile URL, Instagram Handle, Instagram Followers, Spotify Monthly Listeners, Geographic Region, Popularity Score.
   - Rows: 330 artists (after all filtering applied).
   - Use Case: Artist outreach prioritization, database seeding, analytics.

3. **`spotify-scraper` GitHub Repository** – Complete source code, documentation, and configuration.
   - Includes: Both Python scripts, requirements.txt, comprehensive README, .gitignore.
   - Use Case: In-house team can execute future artist discovery runs, modify parameters, or integrate into CI/CD pipelines.
   - Maintenance: Future consultants or Jamtown engineers can fork/clone and maintain independently.

4. **Apify Configuration Documentation** – Step-by-step guide for running Instagram enrichment scraper.
   - Includes: Account setup, API key management, parameter settings, cost estimation, output handling.
   - Use Case: Client can re-run Instagram enrichment on new artist URL batches as discovery scales.

5. **Outreach Template & Campaign Guide** – Instagram DM template, Chrome extension setup instructions, rate-limit best practices, and trust-building recommendations.
   - Use Case: Reproducible, scalable artist outreach process.

---

## 4. Integration with AI & Database Platform

### 4.1 Data Flow to Framer Database & AI Workstream

**Workflow:**

1. **Artist Outreach:** Instagram DM sent to qualified artist with signup link.
2. **Artist Response:** Artist clicks link and completes profile signup form on Framer website (`artist-contact` form).
3. **Profile Data Capture:** Artist submits bio, performance genres, preferred causes, availability.
4. **Database Ingestion:** Framer captures form submission and stores artist profile in backend database.
5. **AI Processing:** AI workstream ingests artist profiles (bio, causes, genres, performance history) and:
   - Vectorizes artist preferences and bio data.
   - Creates embeddings for matching against nonprofit event opportunities.
   - Enables real-time artist-to-event matching.

### 4.2 Data Schema Requirements

**Artist Profile Fields (Captured via Framer Form & AOS Pipeline):**
- Spotify Profile URL
- Instagram Handle & Follower Count
- Artist Name
- Genre(s)
- Bio / Artist Statement
- Causes/Values Alignment (e.g., social justice, environmental, education)
- Geographic Availability (willing to perform in which regions?)
- Monthly Listeners (Spotify metric; used for audience size verification)
- Contact Email (optional)

**Data Quality Expectations:**
- All mandatory fields populated before artist profile considered "active" in database.
- Duplicate profiles flagged and merged.
- Inactive accounts (no login in 90 days) marked for outreach re-engagement.

---

## 5. Challenges, Learnings & Optimization

### 5.1 Key Challenges Encountered

#### Challenge 1: Limited Artist Information from Official Spotify API
- **Problem:** Spotify API provides only basic metadata (follower count, genres); no contact info, bio, or Instagram profiles.
- **Impact:** Unable to build rich artist profiles or find direct contact channels for outreach.
- **Resolution:** Pivoted to hybrid approach: scrape Spotify artist URLs (public data), then enrich with Instagram profiles via Apify.

#### Challenge 2: Ethical & Legal Constraints of Web Scraping
- **Problem:** Direct scraping of Spotify website violates Terms of Service; raises data ownership and privacy concerns.
- **Impact:** Risked account suspension, legal liability, and reputational damage.
- **Resolution:** Decided to scrape only public, static artist profile URLs (not protected data); avoided extracting Spotify-specific proprietary metrics. Used third-party Apify service for Instagram enrichment to further reduce direct scraping liability.

#### Challenge 3: Artist Trust & Spam Perception
- **Problem:** Cold Instagram DMs from unknown platforms are common spam/scam vectors in music industry; artists justifiably skeptical.
- **Impact:** Risk of low response rates, artist blocking, or platform flagging for spam.
- **Mitigation Strategies Deployed:**
  - Crafted professional, concise message emphasizing value (free platform, nonprofit focus).
  - Provided clear call-to-action (direct link to signup).
  - Planned social proof (LinkedIn verification, artist testimonials on website).
  - Follow-up via email with team info and credentials to verify legitimacy.
  - Monitor early response rates and adjust messaging if needed.

#### Challenge 4: Outreach Capacity Limitations (Free Chrome Extension Tier)
- **Problem:** Free-tier Chrome extension limits to ~210 DMs/day; reaching 330+ artists at scale requires weeks/months.
- **Impact:** Slow customer acquisition velocity; delayed data collection for AI workstream.
- **Resolution Options:**
  - **Short-term:** Accept longer campaign duration; focus on message quality and response rate optimization.
  - **Medium-term:** Upgrade to Pro plan for higher DM capacity.
  - **Long-term:** Evaluate alternative outreach platforms (e.g., Zapier automations, dedicated outreach SaaS tools).

---

### 5.2 Lessons Learned & Best Practices

1. **API Limitations Require Creative Solutions:** Official platform APIs rarely provide all required data; hybrid approaches combining multiple data sources (Spotify + Instagram + Apify) unlock richer, more actionable datasets.

2. **Ethical Data Collection Builds Trust:** Avoiding direct ToS violations and respecting artist privacy strengthens long-term platform credibility, even if it requires additional infrastructure or cost.

3. **Automation + Human Messaging = Higher Engagement:** Automated DMs at scale must include personalization and transparent value props to overcome spam skepticism; generic mass messaging fails.

4. **Iterative Testing Validates Assumptions:** The Apify pilot (330-artist cohort) revealed realistic yield rates (30% Instagram match rate) and audience distribution; full-scale deployment will benefit from this ground truth.

5. **Distributed Scraping Improves Resilience:** Running multiple parallel workers (Playwright contexts) provides fault tolerance and accelerates data collection compared to single-threaded crawling.

---

## 6. Ongoing Operations & Handover

### 6.1 What the Client Owns & Can Execute Independently

The client now possesses:

1. **`spotify-scraper` GitHub Repository** – Complete, documented codebase for future artist discovery runs.
   - Can execute: `python discover_artists.py --count <N> --fresh` to discover new artist cohorts.
   - Can modify: Audience thresholds, geographic filters, query diversity parameters.

2. **Apify Configuration & Instructions** – Step-by-step guide to run Instagram enrichment on new artist URL batches.
   - Can execute: Upload new URLs to Apify, run enrichment scraper, download JSON outputs.
   - Can modify: Apify parameters and pricing tier based on volume/budget.

3. **Chrome Extension Outreach Bot Setup** – Instructions for deploying automated Instagram DM campaigns.
   - Can execute: Load message template, input artist usernames, schedule DM campaign.
   - Can modify: Message template, timing between DMs, target cohorts.

4. **Filtered Artist Dataset** – 330 qualified artists with Instagram handles and Spotify data; ready for outreach.
   - Can execute: Begin DM outreach immediately or re-segment by genre/region as needed.

### 6.2 Monitoring & Success Metrics (Ongoing)

As the outreach campaign progresses, the client should track:

- **Response Rate (%):** % of DMs that receive a reply within 7 days of send.
- **Sign-Up Conversion Rate (%):** % of respondents who complete artist profile signup on Framer.
- **Database Entries Per Week:** # of new, validated artist profiles added to backend database per week.
- **Geographic Diversity:** # of artists by state/region; ensure balanced national coverage.
- **Message Refinement:** Document artist feedback, objections, and questions; refine template if response rate stalls.

**Reporting Cadence:** Weekly tracking; monthly summary and optimization review.

---

### 6.3 Future Scaling & Next Steps

#### Short-Term (Next 2–4 Weeks)
- Complete outreach to first 330 qualified artist cohort.
- Monitor response rate and sign-up conversion.
- Refine message template based on initial feedback.
- Document early learnings for follow-up cohorts.

#### Medium-Term (Weeks 5–8)
- Execute second artist discovery run: discover and filter next 50,000+ artists.
- Run Apify enrichment on expanded cohort.
- Deploy second wave of DM outreach.
- Evaluate Pro-tier Chrome extension upgrade ROI based on current campaign velocity.

#### Long-Term (Month 3+)
- Establish feedback loop between outreach response data and AI workstream.
- Refine audience targeting based on which artist segments drive highest signup rates and data quality.
- Consider alternative outreach channels (email outreach to non-respondents, influencer seeding, organic social).
- Evaluate dedicated music industry outreach platforms or partnership approaches.

---

## 7. Appendices

### Appendix A: Technical Installation & Setup Guide

**Environment Setup (Client/Future Engineers):**

```bash
# Clone repository
git clone [spotify-scraper-repo-url] jamtown-spotify-scraper
cd jamtown-spotify-scraper

# Create Python virtual environment
python3 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt

# Install Playwright browser
playwright install
```

**Running the Scraper:**

```bash
# Discover 100,000 artist URLs (first run)
python discover_artists.py --count 100000 --fresh

# Resume prior discovery run (append to existing urls.txt)
python discover_artists.py --count 50000

# Analyze Apify JSON exports and generate artist_popularity.csv
python analyze_artists.py "JSON Files"
```

### Appendix B: Apify Configuration & Execution Steps

1. **Create Apify Account** & subscribe to Instagram Artist Scraper actor.
2. **Upload Artist URLs** – Paste Spotify artist profile URLs into Apify input (one per line).
3. **Configure Scraper Parameters:**
   - Extract: Instagram handle, follower count, bio, website, email (if available).
   - Timeout: 30 seconds per profile.
   - Proxy rotation: Enable to avoid Instagram rate-limits.
4. **Execute Run** – Apify will scrape all profiles; cost ~$0.007 per profile.
5. **Download JSON Export** – Save raw output to `JSON Files/` directory.
6. **Run Analyzer** – Execute `python analyze_artists.py "JSON Files"` to filter and rank results.

### Appendix C: Chrome Extension Outreach Campaign Setup

1. **Install Extension:** [Automated Instagram DM Bot - Chrome Web Store Link]
2. **Load Message Template:**
   ```
   Hi <Username>! I'm reaching out from Jamtown, where we connect artists
   with nonprofits hosting events and performances. We can help you get matched
   with events that fit your genre and schedule, grow your reach by performing
   for new audiences, and handle all the logistics! Moreover, we will provide
   you with a stage to perform at for causes that you care about. It's
   completely free, we'd love it if you could check it out and join our
   artist community here: https://traditional-reason-961104.framer.app/artist-contact
   ```
3. **Prepare Artist List:** Export Instagram usernames from `artist_popularity.csv` into a text file (one username per line).
4. **Load into Extension:** Paste usernames into extension input; set rate limit to 5–7 minutes between DMs.
5. **Monitor:** Track outgoing DMs; pause if Instagram account receives warnings or elevated spam score.
6. **Review Responses:** Check DM inbox daily for responses; log respondent data and conversation quality.

### Appendix D: Data Schema & CSV Column Reference

**artist_popularity.csv Columns:**

| Column | Data Type | Description |
| --- | --- | --- |
| artist_name | String | Artist display name (from Spotify) |
| spotify_url | String | Direct link to artist's Spotify profile |
| instagram_handle | String | Artist's Instagram username (from Apify) |
| instagram_followers | Integer | Instagram follower count (snapshot from enrichment run) |
| spotify_monthly_listeners | Integer | Monthly listener count (from Spotify API) |
| geographic_region | String | US state or region (inferred from profile location data) |
| popularity_score | Float | Normalized popularity ranking (0–1); combines follower/listener metrics |
| genre_primary | String | Primary music genre (from Spotify) |
| genre_secondary | String | Secondary genres (from Spotify) |
| contact_email | String | Email address if available (rare; often PR email) |
| data_collection_date | Date | Date of Apify enrichment run (YYYY-MM-DD) |

### Appendix E: Troubleshooting & Common Issues

**Issue: `discover_artists.py` stalls or crashes**
- **Solution:** Increase `--scroll-pause-ms` parameter (e.g., 500–1000ms) to reduce load on Spotify and avoid detection.
- Check network connection and Spotify service status.
- Reduce `--workers` count if local machine runs out of memory.

**Issue: Apify Instagram scraper returns low yield (~20–30%)**
- **Expected:** This yield rate is normal; many artists lack linked Instagram profiles or have inactive accounts.
- **Solution:** Accept as baseline; focus on high-quality responses from the 30% matched profiles.

**Issue: Chrome extension Instagram DM bot account flagged or suspended**
- **Cause:** Excessive DM volume within short timeframe.
- **Solution:** Increase rate-limit to 10–15 minutes between DMs; consider using multiple accounts in rotation.
- **Prevention:** Never send > 500 DMs/day per account; space campaigns over 2+ weeks.

**Issue: Artist responses indicating trust concerns or spam skepticism**
- **Solution:** Prepare follow-up email template with company verification, social proof, and team credentials.
- Include link to artist testimonials or press coverage on website.
- Offer direct phone call or video chat to verify legitimacy.

---

## 8. Conclusion

The **Automated Outreach Strategy** workstream successfully developed a scalable, ethical, and efficient artist acquisition pipeline that transforms Jamtown's customer acquisition from manual, low-throughput effort to systematic, automated discovery and engagement at scale. By combining custom Spotify artist scraping with third-party Instagram enrichment and automated DM outreach, the workstream has generated a qualified 330-artist cohort ready for immediate engagement and has provided the client with reusable, maintainable infrastructure for future discovery and outreach cycles.

The pipeline directly feeds the Jamtown database and AI matching workstream with fresh artist data, enabling the platform to deliver on its core promise: connecting independent artists with meaningful, cause-aligned performance opportunities. With ongoing monitoring of response rates, sign-up conversions, and database growth, the client can iteratively refine both the artist targeting and outreach messaging to maximize acquisition velocity and data quality.

The handover of the `spotify-scraper` GitHub repository, Apify documentation, and outreach campaign toolkit ensures the client can operate and scale this workstream independently, while maintaining the option to engage future consultants for optimization, platform integration, or broader ecosystem expansion.

### Final Operational Note

While the automated outreach pipeline is highly **low-effort** and requires minimal ongoing maintenance, the practical outcomes showed a **very low success rate in generating genuine artist connections**. Only a small fraction of contacted artists responded or proceeded to the sign-up stage. Additionally, although inexpensive at small scale, the **Apify enrichment costs accumulate quickly** as the number of processed artist profiles grows, and should be monitored closely if future runs are performed.


---

**Document Version:** 2.0  
**Last Updated:** November 30, 2024  
**Status:** **Complete**