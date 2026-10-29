---
cssclasses:
  - cards
  - cards-cols-3
---


# Systems

Computer Systems curriculum: 57 lessons, 8 levels, worked in order. Lessons are created on demand — say "Create lesson ___ for computer systems" and the lesson is created in its Level folder, with nav, using [[Lesson Template]] and [[Learning Philosophy]].

← Previous: — | ↑ Parent: [[Learning]] | Next: — →

## Levels

- [[Level 01 - Foundations]] — Lessons 01–06
- [[Level 02 - Hardware Foundations]] — Lessons 07–15
- [[Level 03 - Program Execution]] — Lessons 16–23
- [[Level 04 - Memory]] — Lessons 24–33
- [[Level 05 - Operating Systems]] — Lessons 34–42
- [[Level 06 - Modern Computing]] — Lessons 43–49
- [[Level 07 - Security Foundations]] — Lessons 50–56
- [[Level 08 - Synthesis]] — Lesson 57

## All Systems lessons created so far

```dataview
TABLE WITHOUT ID
  file.link AS "Lesson",
  choice(status = "done", "✓ Done", choice(status = "in-progress", "● In progress", "○ Not started")) AS "Status"
FROM "Learning/Systems"
WHERE type = "lesson"
SORT lesson ASC
```

## Curriculum outline

See [[Systems Curriculum]] for the full 57-lesson list. Lesson 45, Networking Foundations, links to the existing [[Networking]] notes 00–18 instead of rebuilding them. Lesson 46 builds sockets and servers on top.
