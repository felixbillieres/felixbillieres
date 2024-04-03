# 🏉 Social Media OSINT

Twitter Advanced Search - [https://twitter.com/search-advanced](https://twitter.com/search-advanced)

<figure><img src="../../../../.gitbook/assets/image (344).png" alt=""><figcaption></figcaption></figure>

### Twitter OSINT <a href="#lecture_heading" id="lecture_heading"></a>

**Geocode Twitter Search Step-by-Step Documentation:**

**STEP 1: Determine the Geographical Location**

* **Objective:** Identify the latitude and longitude coordinates for the location you want to search for tweets.
* **Services:** Utilize services like Google Maps or geocode.xyz for accurate coordinates.
* **Example:** For New York:
  * Latitude: 40.712776
  * Longitude: -74.005974
* **Tip:** If using Google Maps, find coordinates in the URL.

**STEP 2: Choose the Radius**

* **Objective:** Define the search radius around the chosen location.
* **Example:** Select a radius, e.g., 10km (\~6 miles) from the location.
* **Consideration:** Twitter's geocode radius is specified in kilometers.

**STEP 3: Compose the Search String**

* **Objective:** Formulate a search string for tweets by location.
* **Format:** `geocode:latitude,longitude,radius`
* **Example:** `geocode:40.712776,-74.005974,10km`

**STEP 4: Implement the Search String**

* **Objective:** Integrate the geocode string with your desired keyword for Twitter search.
* **Example:** `oscars geocode:40.712776,-74.005974,10km`
* **Platform:** Use TweetBinder or other tools supporting geocode searches.
* **Note:** Results are subject to user privacy settings; not all tweets may be accessible.

**Twitter Analysis Tools Documentation:**

**1. Social Bearing**

* **Website:** [Social Bearing](https://socialbearing.com/)
* **Functionality:**
  * Social Bearing provides in-depth analysis and visualization of Twitter data.
  * Features include sentiment analysis, hashtag tracking, and user engagement metrics.
  * Useful for understanding trends and user behaviors on Twitter.

**2. Twitonomy**

* **Website:** [Twitonomy](https://www.twitonomy.com/)
* **Key Features:**
  * Comprehensive Twitter analytics and insights.
  * Provides detailed information about tweets, retweets, mentions, and hashtags.
  * Allows tracking of competitors and influencers.
  * Visualizes user interactions and engagement.

**3. Sleeping Time**

* **Website:** [Sleeping Time](http://sleepingtime.org/)
* **Purpose:**
  * Sleeping Time analyzes Twitter activity patterns to estimate when a user is likely to be asleep.
  * Useful for scheduling tweets or analyzing the most active time of a Twitter user.

**4. Mentionmapp**

* **Website:** [Mentionmapp](https://mentionmapp.com/)
* **Functionality:**
  * Mentionmapp visualizes Twitter conversations by mapping out relationships between users.
  * Shows connections, mentions, and conversations in an interactive graph.
  * Useful for understanding the network and relationships within a Twitter discussion.

**5. Tweetbeaver**

* **Website:** [Tweetbeaver](https://tweetbeaver.com/)
* **Key Features:**
  * Tweetbeaver offers various Twitter analytics, including tweet analysis and hashtag tracking.
  * Provides insights into user engagement and content performance.
  * Useful for monitoring the effectiveness of tweets and hashtag campaigns.

**6. Spoonbill.io**

* **Website:** [Spoonbill.io](http://spoonbill.io/)
* **Purpose:**
  * Spoonbill.io is a Twitter analytics tool that focuses on user profiling.
  * Analyzes user activity, interests, and engagement patterns.
  * Useful for audience segmentation and targeted content creation.

**7. Tinfoleak**

* **Website:** [Tinfoleak](https://tinfoleak.com/)
*   **Key Features:**

    * Tinfoleak is a powerful OSINT tool for Twitter.
    * Provides detailed information about a Twitter user, including geolocation, devices used, and hashtag analysis.
    * Useful for investigative purposes and understanding a user's online footprint.



### Twitter Dorks

```
# Tweets containing word1 AND word2 (default operator)
<word1> <word2>

# Containing exact expression
"word1"

# Tweets containing word1 OR word 2
<word1> OR <word2>

# Containing "cyber" but without "security"
cyber -security

# All tweets from an account or responding to an account
from:<account>
to:<account>

# Filter results and display only results from followed accounts
filter:follows
exclude:medias

# Tweets after the date or before the date
<word> since:2015-02-20
<word> until:2015-02-20

# Minimal RT / likes / replies
min_retweets:x
min_faves:x
min_replies:x

# Language
lang:fr

# Tweets positive and negative
<word> :)
<word> :(

# Tweets with a question
<word> ?

# Location (city) with a distance range
near:Paris within:25km

```

### Facebook OSINT <a href="#lecture_heading" id="lecture_heading"></a>

1. **Sowdust Github**
   * **Website:** [Sowdust Github - fb-search](https://sowdust.github.io/fb-search/)
   * **Functionality:**
     * Sowdust's fb-search tool is designed for conducting advanced Facebook searches.
     * Enables users to search for people, posts, photos, and more on Facebook.
     * Provides detailed results, including profile information, groups, and events.
2. **IntelligenceX Facebook Search**
   * **Website:** [IntelligenceX Facebook Search](https://intelx.io/tools?tab=facebook)
   * **Key Features:**
     * IntelligenceX offers a comprehensive Facebook search tool as part of its OSINT services.
     * Allows users to search for Facebook profiles, posts, and other content.
     * Provides insights into Facebook data that may not be easily accessible through standard search methods.

### Instagram OSINT <a href="#lecture_heading" id="lecture_heading"></a>

[https://github.com/Datalux/Osintgram](https://github.com/Datalux/Osintgram)
