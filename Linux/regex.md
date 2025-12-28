# Regular Expressions (Regex) for Beginners

A **Regular Expression** (Regex) is a powerful sequence of characters that forms a search pattern. You can use it to find, edit, or manipulate text and data. Think of it as a "Find and Replace" tool on steroids.

---

## 1. The Core Components

Regex uses special characters (metacharacters) to represent different types of logic.

### Literal Characters

These are the simplest forms of regex. They match exactly what you type.

* `abc` matches the string "abc".

### Anchor Tags

Anchors don't match characters; they match **positions**.

* `^` (Caret): Matches the **beginning** of a line.
* `$` (Dollar): Matches the **end** of a line.

### Character Classes

These allow you to match a specific set of characters.

* `[abc]`: Matches any one of the characters (a, b, or c).
* `[a-z]`: Matches any lowercase letter from a to z.
* `[0-9]`: Matches any single digit.
* `[^abc]`: (Negation) Matches any character **except** a, b, or c.

---

## 2. Predefined Shorthand Classes

Most modern languages provide shortcuts for common character sets:

| Shortcut | Matches | Equivalent to |
| --- | --- | --- |
| `\d` | Any Digit | `[0-9]` |
| `\w` | Word character (letter, number, or underscore) | `[a-zA-Z0-9_]` |
| `\s` | Whitespace (space, tab, newline) | `[ \t\r\n\f]` |
| `.` | Any character except a newline | N/A |

> **Pro Tip:** Capitalizing the shortcut usually does the opposite. For example, `\D` matches anything that is **not** a digit.

---

## 3. Quantifiers

Quantifiers define **how many** times the preceding character should occur.

* `*` : 0 or more times.
* `+` : 1 or more times.
* `?` : 0 or 1 time (makes it optional).
* `{n}` : Exactly *n* times.
* `{n,}` : *n* or more times.
* `{n, m}` : Between *n* and *m* times.

---

## 4. Logical Operators

* `|` (Pipe): Acts like an **OR** operator. `cat|dog` matches "cat" or "dog".
* `()` (Parentheses): Used for **Grouping**. For example, `(ha)+` matches "ha", "haha", "hahaha", etc.

---

## 5. Putting it Together: Examples

### Matching a Phone Number

If you want to match a pattern like `123-456-7890`:
`\d{3}-\d{3}-\d{4}`

* `\d{3}`: Three digits.
* `-`: A literal hyphen.
* `\d{3}`: Three digits.
* `-`: Another literal hyphen.
* `\d{4}`: Four digits.

### Matching a Basic Email

`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`

* `[a-zA-Z0-9._%+-]+`: One or more characters allowed in usernames.
* `@`: A literal "@" symbol.
* `\.`: A literal dot (we use `\` to "escape" it because `.` usually means "any character").
* `[a-zA-Z]{2,}`: Two or more letters for the domain extension (like .com or .org).

---

## Cheat Sheet Summary

* **`.`** → Anything
* **`\d`** → Digit
* **`\w`** → Word
* **`+`** → 1 or more
* **`*`** → 0 or more
* **`?`** → Optional
* **`^`** → Start
* **`$`** → End

Would you like me to create a set of practice exercises or explain how to use Regex specifically in a language like Java?