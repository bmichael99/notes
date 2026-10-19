---
pm-task: true
projectId: "h1b5urdamsgdifdv"
parentId:
id: "etax4c8xmst6nus7"
title: "Optimistic Error Handling"
type: "task"
status: "todo"
priority: "low"
start: "2026-08-14"
due: ""
progress: 0
assignees: []
tags: []
subtaskIds: []
dependencies: []
createdAt: "2026-08-14T16:50:25.302Z"
updatedAt: "2026-08-14T17:08:28.356Z"
---

Optional optimistic error handling, will need to refactor the way errors are handled and listed. Not worth doing imo

You would use the event id that's passed into the edit page that allows for the error to be displayed. Resolve that event and then append a new error with the errors stripped from the one that's updated.

Project: [[Labeling|Labeling]]