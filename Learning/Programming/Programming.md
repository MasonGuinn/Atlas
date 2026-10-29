---
cssclasses:
  - cards
  - cards-cols-3
---


# Programming

← Previous: — | ↑ Parent: [[Learning]] | Next: — →

## Tracks

- C++ — notes go in `Learning/Programming/C++/`
- Python — notes go in `Learning/Programming/Python/`

A separate C++ curriculum is planned, triggered the same way as Systems: "Create lesson ___ for c++". It does not exist yet — do not mix it with the Systems curriculum.

```dataview
TABLE WITHOUT ID
  file.link AS "Lesson",
  choice(status = "done", "✓ Done", choice(status = "in-progress", "● In progress", "○ Not started")) AS "Status"
FROM "Learning/Programming"
WHERE type = "lesson"
SORT lesson ASC
```
