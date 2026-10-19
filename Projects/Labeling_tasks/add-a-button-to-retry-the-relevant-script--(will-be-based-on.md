---
pm-task: true
projectId: "h1b5urdamsgdifdv"
parentId:
id: "lum7dz4fmsqpl6vq"
title: "Add a button to retry the relevant script? (Will be based on the current flags, we'll have that mapped to the front-end so it knows what script to run). Will need to hit a backend endpoint for this. Maybe after editing something this should happen automatically."
type: "task"
status: "todo"
priority: "medium"
start: "2026-08-12"
due: ""
progress: 0
assignees: []
tags: []
subtaskIds: []
dependencies: []
createdAt: "2026-08-12T23:16:55.190Z"
updatedAt: "2026-08-13T18:42:24.490Z"
---

If the script is already running then have a toast or message on screen that tells the user that the script is already running, if it's not already running than show a notification that the script has started. We don't want to kick off a cron job if the script is already running, make sure we have fail safes for the user spam clicking the button or double clicking it

Project: [[Labeling|Labeling]]