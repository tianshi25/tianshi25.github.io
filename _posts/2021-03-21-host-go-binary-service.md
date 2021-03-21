---
layout: post
title: "Host Go Binary Service"
date:  2021-03-21 22:17:11
categories: go
tags: go docker
---
Host additional check, a go binary I release as web service.





* content
{:toc}
# Requirements

I maitains [AdditionalCheck](https://github.com/tianshi25/additionalCheck), an open source code style checker I developed for my team. To promotes its usage, I decided to convert it to a web service.

User can submit a Gerrit link to the web service. Service will later provide a web page of code check results.

# Design


## UI

  * Main Page

    * Interface to submit coding request
    * table of previous request
      * support search
      * request status
      * link to result
      * link to execute log

  * Result Page

    * View result with color

    * Button to download result

## Flow

1. Parse user submits gerrit link, download corresponding code.
   * Use gerrit REST api to get repo info
   * Use git to download the code
2. Run **Additional Check**, get results.
   * Store the original result
3. Convert results to a web page

## Other Functions

* Clear outdated record.
* Run service in docker