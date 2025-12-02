
# Adding a new post

To add a new post to the site

- you mush first move the markdown file into the `Posts` directory
- Do a git add
- Commit and  Push

> Best to do one post at a time

> Webhook event is configured to hit site server and fetch post details

## Post markdown must follow format

```
---
title: "<Post Title> "
slug: "<unique slug for post>"
date: "Date"
tags: [one word tags]
---

<<<Content here>>>

```

## Adding a new project

To add a new project to the site

- you mush first move the json file into the `projects` directory
- Do a git add
- Commit and  Push

> Best to do one project at a time

> Webhook event is configured to hit site server and fetch project details

## Project json format

```
{
  "Name": "",
  "Description": "",
  "Url": "",
  "Image": ""
}
```
