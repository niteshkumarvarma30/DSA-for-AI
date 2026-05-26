# Object and their Blue-Prints

This screenshot perfectly illustrates exactly what we just discussed about objects and their blueprints (classes)!

The errors you are seeing happen because you are trying to use a method that belongs to one type of object blueprint on a completely different type of object. Python checks the object's type field, realizes the requested behavior doesn't exist for that class, and throws an `AttributeError`.

Let's break down both cases from your image to see exactly what Python is doing under the hood.

---

### 🔍 Case 1: `L = [1, 2, 3]` followed by `L.upper()`

* **What you did:**
  You created a list `L` and tried to convert it to uppercase.
* **Why Python throws `AttributeError: 'list' object has no attribute 'upper'`:**
  When you define `L = [1, 2, 3]`, Python creates a memory block for a `list` object.
  The `list` class blueprint contains methods like `.append()`, `.pop()`, and `.sort()`. It has absolutely no idea what `.upper()` means because uppercase is a text concept, not a collection concept.
  When you call `L.upper()`, Python looks at the blueprint for lists, doesn't find a function named `upper`, and stops you immediately.

---

### 🔍 Case 2: `s = 'hello'` followed by `s.append('x')`

* **What you did:**
  You created a string `s` and tried to append a character to it.
* **Why Python throws `AttributeError: 'str' object has no attribute 'append'`:**
  When you define `s = 'hello'`, Python creates a memory block for a `str` (string) object.
  The `str` class blueprint contains text-processing methods like `.upper()`, `.lower()`, and `.split()`. It does not contain `.append()`.
  
  Furthermore, strings in Python are **immutable** (they cannot be modified in-place once created). If you want to add something to a string, the `str` blueprint requires you to use concatenation (`+`) to create a brand new string object instead:
  
  ```python
  s = 'hello'
  s = s + 'x'  # This works! Creates a new string 'hellox'
  ```

---

### 💡 The Core Takeaway

Every object in Python is restricted to the tools provided by its own specific blueprint:

| Object | Its Blueprint Class | Allowed Methods (Examples) | Missing Methods (Will Crash) |
| :--- | :--- | :--- | :--- |
| `[1, 2, 3]` | `list` | `.append()`, `.pop()`, `.extend()` | `.upper()`, `.lower()` ❌ |
| `'hello'` | `str` | `.upper()`, `.split()`, `.replace()` | `.append()`, `.pop()` ❌ |
