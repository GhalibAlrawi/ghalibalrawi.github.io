---
layout: post
title: My First Python Script
---

My first Python script calculated the circumference of a circle using Pi. It takes the diameter of the circle through user input via console, multiplies it by Pi to get circumference, and prints it out.
```python
from math import pi

try:
    radius = float(input("Input the radius of the circle : "))

except ValueError:
    print("Invalid input, enter a number.")

else:
    print("The area of a circle with the radius " + str(r) + " is: " + str(pi * r**2))
```