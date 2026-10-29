---
cssclasses:
  - dashboard
  - cards
  - cards-cols-3
  - table-wide
---

# Atlas

Your learning home. Everything below updates itself — change a lesson's `status` in its frontmatter, and this page, the track indexes, and the progress cards update with it.

## Tracks

- **[[Learning/Networking|Networking]]** — 19 notes, 00–18, complete
- **[[Learning/Systems|Systems]]** — 57 lessons, 8 levels, in progress
- **[[Learning/Programming|Programming]]** — C++ and Python
- **[[Learning/Cybersecurity|Cybersecurity]]** — Security+ and cybersecurity
- **[[School]]** — CSIA 320, CSIA 420, SOC 305

## Continue

Last-edited lessons, newest first. This is your resume point.

```dataview
TABLE WITHOUT ID
  file.link AS "Lesson",
  choice(status = "done", "✓ Done", choice(status = "in-progress", "● In progress", "○ Not started")) AS "Status",
  domain AS "Track"
FROM "Learning"
WHERE type = "lesson"
SORT file.mtime DESC
LIMIT 9
```

## Next up

The next not-started lesson in each track, in order.

```dataview
TABLE WITHOUT ID
  file.link AS "Lesson",
  domain AS "Track",
  lesson AS "No."
FROM "Learning"
WHERE type = "lesson" AND status = "not-started"
SORT domain ASC, lesson ASC
LIMIT 9
```

## In progress

```dataview
LIST WITHOUT ID
  file.link + " — " + domain
FROM "Learning"
WHERE type = "lesson" AND status = "in-progress"
SORT domain ASC, lesson ASC
```

## Progress

```dataview
TABLE WITHOUT ID
  domain AS "Track",
  length(filter(rows, (r) => r.status = "done")) AS "Done",
  length(rows) AS "Total"
FROM "Learning"
WHERE type = "lesson"
GROUP BY domain
SORT domain ASC
```
