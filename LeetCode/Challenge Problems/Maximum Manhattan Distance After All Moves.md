# Maximum Manhattan Distance After All Moves

## The Concept
The problem asks us to find the maximum possible **Manhattan Distance** from the origin `(0, 0)` after performing a series of moves. We are given standard directional moves (`U`, `D`, `L`, `R`) and wildcard moves (`_`) that we can convert into any of the four directions.

The formula for Manhattan distance between the origin and a point $(x, y)$ is: 
$\text{Distance} = |x| + |y|$

To solve this optimally, we must realize two mathematical properties:

### 1. Axes are Independent
Moving horizontally (`L`, `R`) has absolutely no impact on your vertical position, and moving vertically (`U`, `D`) has no impact on your horizontal position. 
Therefore, we can calculate our guaranteed net movement on each axis separately:
* **Net Horizontal Distance:** $|R - L|$
* **Net Vertical Distance:** $|U - D|$

### 2. Wildcards Always Help
Because we want to *maximize* our distance, we will always assign a wildcard (`_`) to whichever direction pushes us further away from the origin. 
* If we are currently at `(-3, 4)`, we want to move further Left or further Up. 
* We don't even need to know *which* direction we pick; we just know that every single wildcard can be used to optimally stretch our distance by exactly `1` unit.

### The Final Formula
Instead of simulating the path step-by-step, the maximum distance is simply the sum of our net movements plus the total number of wildcards:
$\text{Max Distance} = |R - L| + |U - D| + \text{count}(\text{'_'}) $

---

## The Code (Python)

```python
class Solution:
    def maxDistance(self, moves: str) -> int:
        # 1. Count the occurrences of each guaranteed direction
        up = moves.count('U')
        down = moves.count('D')
        left = moves.count('L')
        right = moves.count('R')
        
        # 2. Count the available wildcards
        wildcards = moves.count('_')
        
        # 3. Calculate the absolute net distance on both axes
        net_x = abs(right - left)
        net_y = abs(up - down)
        
        # 4. Add the wildcards to the guaranteed distance
        return net_x + net_y + wildcards
```

## Complexity Analysis

### Time Complexity: $\mathcal{O}(N)$

Where $N$ is the length of the string `moves`.

The Python `.count()` method iterates through the string. Even though we call it 5 separate times, it runs in linear time, resulting in $\mathcal{O}(5N)$, which simplifies to:

$$
\mathcal{O}(N)
$$

### Space Complexity: $\mathcal{O}(1)$

We are only storing a few integer variables:

* `up`
* `down`
* `left`
* `right`
* `wildcards`
* `net_x`
* `net_y`

This requires a constant amount of auxiliary memory regardless of how large the input string becomes.

Therefore, the space complexity is:

$$
\mathcal{O}(1)
$$
