<div>
<img align="left" src="https://images.weserv.nl/?url=avatars.githubusercontent.com/u/768070?v=4&h=100&w=100&fit=cover&mask=circle&maxage=7d" alt="Profile picture" width="50" height="50" hspace="10">

<div>
Hi :wave: My name is Anatoli and this is a template for a portfolio project on <a href="https://www.sqlhabit.com">SQL Habit</a>. It has a collection of dashboards for a fictional startup named Bindle, a subscription-based book service.
</div>
</div>

<br>

### Table of Contents
- [AARRR metrics for a subscription-based book app](#aarrr-metrics-for-a-subscription-based-book-app)
  - [Acquisition](#acquisition)
    - [Daily signups](#daily-signups)
    - [Daily organic vs paid signups](#daily-organic-vs-paid-signups)
    - [Biggest marketing channels](#biggest-marketing-channels)
  - [Activation](#activation)
    - [Soft-activation or users with mobile app](#soft-activation-or-users-with-mobile-app)
    - [Where users come from?](#where-users-come-from)

# AARRR metrics for a subscription-based book app

AARRR is a collection of metrics to measure any business:

* Acquisition
* Activation
* Retention
* Revenue
* Referral

:iphone: In this project, I'm working with a data warehouse of a fictional startup named Bindle. Bindle has a mobile and a web apps for reading books. It charges users monthly or yearly subscription fee.

:bar_chart: In the project, we'll build charts with basic AARRR metrics for Bindle.

## Acquisition

Let's see how many users sign up for Bindle daily, where they are coming from, what are the biggest acquisition channels are on web and mobile.

### Daily signups

Although the absolute number of signups might seem a vanity metric, it's nice to visualize it and track our day-to-day growth trends.

Here's the query behind this dashboard:

~~~pgsql
SELECT
  created_at::date AS d,
  COUNT(*) AS user_count
FROM users
WHERE
  -- from now on we'll use this date filter instead of
  -- now() - '30 days'::interval because the dataset
  -- is from 2018
  date_part('year', created_at) = 2018
  AND date_part('month', created_at) = 2
GROUP BY 1
ORDER BY 1 DESC
~~~

<div>
  <img align="left" src="./images/charts/daily_signups.png" alt="Daily signups in the past 30 days" width="75%">

  A number of new user accounts created per day for the past 30 days.
</div>

<br clear="left"/>
<br>

### Daily organic vs paid signups

Let's transform the absolute-number vanity metric into something more actionable. What drives our growth? Let's see the ratio between organic signups and users we acquired through marketing campaigns:

~~~pgsql
SELECT
  created_at::date AS d,
  CASE WHEN utm_medium IS NULL THEN 'organic' ELSE 'paid' END AS user_source,
  COUNT(*) AS user_count
FROM users
WHERE
  date_part('year', created_at) = 2018
  AND date_part('month', created_at) = 2
GROUP BY 1, 2
ORDER BY 1 DESC
~~~

<div>
  <img align="right" src="./images/charts/daily_organic_vs_paid_signups.png" alt="Daily organic vs paid signups" width="75%">

  A number of new organic and paid user accounts created per day for the past 30 days.
</div>

<br clear="right"/>
<br>

### Biggest marketing channels

Let's zoom into paid signups and see which channels and campaigns drive our growth.

~~~pgsql
SELECT
  utm_source,
  utm_campaign,
  utm_content,
  COUNT(*) AS user_count
FROM users
WHERE
  date_part('year', created_at) = 2018
  AND date_part('month', created_at) = 2
  AND utm_medium IS NOT NULL
GROUP BY 1, 2, 3
ORDER BY 4 DESC
~~~

<div>
  <img align="left" src="./images/charts/biggest_marketing_channels.png" alt="Biggest marketing channels" width="75%">

  As you can see, Twitter and Facebook are our primary paid CPC marketing channels.
</div>

<br clear="left"/>
<br>

## Activation

Now we've glimpsed into the user numbers, let's see what happens with these users down the funnel.

### Soft-activation or users with mobile app

Let's see how many users who signed up on the web installed a mobile app.

~~~pgsql
WITH mobile_app_users AS (
  SELECT DISTINCT user_id
  FROM mobile_analytics.events m
)

SELECT
  u.created_at::date AS d,
  CASE WHEN m.user_id IS NOT NULL THEN 'has_mobile_app' ELSE 'only_web' END AS mobile_app_status,
  COUNT(u.id) AS user_count
FROM users u
LEFT JOIN mobile_app_users m
  ON u.id = m.user_id
WHERE
  date_part('year', u.created_at) = 2018
  AND date_part('month', u.created_at) = 2
GROUP BY 1, 2
ORDER BY 1 DESC
~~~

<div>
  <img align="right" src="./images/charts/new_signups_with_mobile_apps.png" alt="New web signups with mobile apps" width="75%">

  Let's look at total number of users daily.
</div>

> [!NOTE]
> Even if new users haven't read a book yet, we may reach them later in the mobile app via a push notification or an in-app message.

<br clear="right"/>
<br>


### Where users come from?

<div>
  <img align="left" src="./images/charts/868.png" alt="Paid vs organic signups" width="75%">

  Number of users who came organically vs via marketing campaigns.
</div>

<br clear="left">
<br>

<div>
  <img align="right" src="./images/charts/1291.png" alt="Number of users per country" width="75%">

  TOP-10 countries with their respective number of users.
</div>

<br clear="right">
<br>

Now we're back to free flowing text.
