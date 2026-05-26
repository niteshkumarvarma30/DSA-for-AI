# Power of OOP - Create Own Data Types

Yes, exactly! That is one of the absolute core powers of Object-Oriented Programming (OOP).

When you use the `class` keyword in Python, you are not just writing code—you are building a brand new, custom data type from scratch. Your custom type will behave exactly like Python's built-in types (`list`, `str`, `int`), complete with its own allowed data attributes and unique methods.

Here is how you do it, building a custom data type step-by-step.

---

## 🏗️ How to Create a Custom Data Type

Let's say we want to create a brand new data type called `Coordinate` to represent a point $(x, y)$ on a map. Python doesn't have a built-in coordinate type, so we will make our own.

### 1. Define the Blueprint (The Class)
We use a special constructor method called `__init__` to define what raw data our type will hold.

```python
class Coordinate:
    # The Constructor: Defines the internal data of our new datatype
    def __init__(self, x_val, y_val):
        self.x = x_val  # Custom attribute
        self.y = y_val  # Custom attribute
```

### 2. Instantiate It (Create the Object)
Now, `Coordinate` can be used just like any other data type to instantiate an object in memory.

```python
# Creating an instance of our new Coordinate data type
point1 = Coordinate(4, 5)

print(type(point1))  
# Output: <class '__main__.Coordinate'>  <-- Look at that! It's a brand new class type.

print(point1.x)      # Output: 4
```

---

## 🚀 Adding "Allowed Behaviors" (Methods)

Just like a `str` datatype has `.upper()` and a `list` has `.append()`, we can give our `Coordinate` type its own unique actions by writing functions inside the class.

Let's add a method to calculate the distance from another point:

```python
import math

class Coordinate:
    def __init__(self, x_val, y_val):
        self.x = x_val
        self.y = y_val
        
    # Custom Method: Only works on Coordinate data types!
    def distance_from_origin(self):
        return math.sqrt(self.x**2 + self.y**2)
        
    # Another Custom Method: Interacting with another coordinate object
    def move_point(self, dx, dy):
        self.x += dx
        self.y += dy

# --- Using our fully-featured custom data type ---
p = Coordinate(3, 4)

# Call our custom method
print(p.distance_from_origin())  # Output: 5.0

p.move_point(1, 1)
print(p.x, p.y)                  # Output: 4 5
```

> [!NOTE]
> If you tried to call `p.append(10)`, Python would throw the exact same `AttributeError` you saw earlier because our custom blueprint doesn't include an `append` method!

---

## 🧠 Dunder Methods: Making Your Type Feel "Built-in"

Python allows you to use special "Double Underscore" (Dunder) methods to change how your custom data type interacts with standard Python math operators like `+`, `-`, or `print` statements.

```python
class Coordinate:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        
    # 1. Controls what happens when you print() this object
    def __str__(self):
        return f"Point({self.x}, {self.y})"
        
    # 2. Controls what happens when you use the '+' operator
    def __add__(self, other):
        return Coordinate(self.x + other.x, self.y + other.y)

# Let's see it in action:
p1 = Coordinate(1, 2)
p2 = Coordinate(10, 20)

# Python converts this '+' directly into p1.__add__(p2) under the hood!
p3 = p1 + p2  

print(p3)  # Output: Point(11, 22)
```

By designing classes this way, you are creating your own structural building blocks. This is exactly how complex data structures like custom Stacks, Queues, Linked Lists, and Graphs are built in Python!
