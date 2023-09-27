# docs

Documentation of the [FreeLeadsData](https://freeleadsdata.com) project.

**Outline:**

1. [Overview](1-overview)
2. [Serialization](2-serialization)
3. [Protocol](3-protocol)
4. [Events](4-events)

## 1. Overview

[FreeLeadsData](https://freeleadsdata.com) is B2B data service that provides the following features:

- Leads Scraping.
- Email Appending & Verification.
- Company Websites Scraping for AI Insights Filtering.
- API for connecting with other scrapers and enrich data.

[FreeLeadsData](https://freeleadsdata.com) is designed for:

1. scaling with no limit by following a micro.services architecture; and
2. reducing costs at minimum.

The general architecture looks like this:

![general architecture](./img/01-general-architecture.png)

1. We use [CockroachDB](https://www.cockroachlabs.com/) for storing data into scalable and serverless database.

2. We use AWS/EC2 for managing a scalable web server for our app. Each AWS/EC2 instance run a FreeLeadsData [app](https://github.com/FreeLeadsData/app). Such an **app** us a fork of [my.saas](https://github.com/leandrosardi/my.saas). 

3. The offline processing is being handled by a micro-service called [micro.data](https://github.com/FreeLeadsData/micro.data). Such a **micro.data** service is a fork of [micro.template](https://github.com/leandrosardi/micro.template).

## 2. Serialization

Any search can be serialized with a `json` descriptor like below.

For a full documentation of search descriptor, refer to this other document:
_(pending)_

```ruby
h = {
    'name' => name,
    'stop_limit' => 999999, # 400 million
    'status' => true,
    'verify_email' => true, 
    'direct_phone_number_only' => false,

    'keywords' => [
        # keywords to include
        { 'value' => '[company_name]', 'type' => 0 },
    ],

    'job_titles' => [
        # job positions to include
        { 'value' => 'Owner', 'positive' => true },
        { 'value' => 'CEO', 'positive' => true },
        { 'value' => 'Founder', 'positive' => true },
        { 'value' => 'President', 'positive' => true },
        { 'value' => 'Director', 'positive' => true },
    ],

    'states' => [
        # locations to include
        { 'value' => 'FL', 'positive' => true }, 
    ],

    'industries' => [
        # industries to include
        { 'value' => 'Real Estate', 'positive' => true },
    ],

    'company_headcounts' => [
        # headcounts to include
        { 'value' => '1 to 10', 'positive' => true },
        { 'value' => '11 to 25', 'positive' => true },
        { 'value' => '26 to 50', 'positive' => true },
    ],

    'company_names' => ['Xerox', 'Google', 'Microsoft'],
}
```

## 3. Protocol

When a **user** create a new search in the **app**, such a search is:

1. assigned to a **micro.data** node; and then
2. pushed to such a **node**.

![FreeLeadsData Protocol 1](./img/02-protocol-1.png)

While the node is 

- **populating** the search (with leads that match with thefilters); and

- **running** other tasks (like **scraping** website, **appending** website insights using AI, **verifying** emails, **exporting** results to a CSV, and **draining** processed results); 

the app may request an update to the node every `x` seconds.

![FreeLeadsData Protocol 1](./img/02-protocol-2.png)

    #'daily_quota' => 400*1000*1000, # 400 million,
    #'credits' => 50,
    #'earning_per_verified_email' => 0.018,

    'auto_drain' => false,
