---
pm-task: true
projectId: "h1b5urdamsgdifdv"
parentId:
id: "6l7e8xedmsrt85ww"
title: "Add edit pages and auto re-run the necessary cron job after update. Add current step awareness and step visibility to frontend for this case and for debugging."
type: "task"
status: "todo"
priority: "medium"
start: "2026-08-13"
due: ""
progress: 0
assignees: []
tags: []
subtaskIds: []
dependencies: []
createdAt: "2026-08-13T17:46:32.048Z"
updatedAt: "2026-08-13T18:28:02.788Z"
---

Two approaches:
1. Allow the user to edit the form directly within our application and when you save the form it will send a post request to logiwa/netsuite and update the information there, then kickoff the relevant cron job

2. Edit buttons go directly to the relevant netsuite/logiwa pages, update that, hit sync and then it will pull the necessary information.

Project: [[Labeling|Labeling]]