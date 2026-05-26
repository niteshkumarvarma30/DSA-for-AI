# Functions Vs Methods

In Python, the difference between a function and a method comes down to one core concept: **association**.

While they both use the `def` keyword and execute a block of code, a function is a standalone entity, whereas a method is a function that belongs to an object or class.

---

### 🏛️ The Main Differences at a Glance

| Feature | Function | Method |
| :--- | :--- | :--- |
| **What is it?** | A standalone block of code defined globally. | A function defined inside a class blueprint. |
| **Association** | Independent; not explicitly tied to any class or object. | Dependent; bound to an instance of an object or class. |
| **How it's called** | Called directly by its name: `my_function()`. | Called on an object using dot notation: `object.my_method()`. |
| **Implicit Argument** | Receives only the arguments you explicitly pass. | Automatically receives the object instance (`self`) as its first argument. |

---

### 🔍 Deep Dive with Code

Let's look at how both look and behave within the same Python script:

```python
# 1. This is a FUNCTION
def calculate_area(length, width):
    return length * width


# 2. This is a CLASS containing a METHOD
class Rectangle:
    def __init__(self, length, width):
        self.length = length
        self.width = width
        
    # This is a METHOD
    def find_area(self):
        return self.length * self.width
```

#### How you call them:

```python
# Calling a Function:
# You pass all data explicitly. It stands completely on its own.
area_func = calculate_area(5, 10)
print(area_func)  # Output: 50


# Calling a Method:
# First, you must instantiate an object.
rect_object = Rectangle(5, 10)

# You call the method ON the object using dot notation.
area_meth = rect_object.find_area()
print(area_meth)  # Output: 50
```

---

### 🧠 The Magic Behind Methods: What is `self`?

Remember how we noted that "everything in Python is an object"? Under the hood, Python translates method calls into function calls behind the scenes!

When you run:
```python
rect_object.find_area()
```

Python takes the object label (`rect_object`) and implicitly passes it as the very first argument into the function definition inside the class blueprint. It effectively executes this:
```python
Rectangle.find_area(rect_object)
```

This is why every instance method you write inside a class must have `self` as its first parameter. `self` is just a variable name pointing right back to the specific object instance that called it, allowing the method to access its internal attributes like `self.length` and `self.width`.

---

### 🛠️ Built-in Examples You Already Know

You've actually been using both of these concepts all along in your DSA coding:
* **Functions:** `len()`, `print()`, `min()`, `max()`. You pass the object into them, like `len(my_list)`.
* **Methods:** `.append()`, `.pop()`, `.upper()`. They are bound to specific data types and invoked using a dot, like `my_list.append(x)`.
