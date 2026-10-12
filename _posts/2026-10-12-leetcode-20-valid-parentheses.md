---
title: "LeetCode #20 — Valid Parentheses"
date: 2026-10-12 11:18:00 +0200
categories: [LeetCode, Python]
tags: [Stack, HashMap, Algorithms, DataStructures]
toc: true
---
# Leet Code #20

## Valid parentheses

https://leetcode.com/problems/valid-parentheses/description/

For this LeetCode series, I'm using Gemini as my coach. For each problem, I follow a structured 1-hour workflow:

- **15 minutes:** Attempting the exercise independently.
- **15 minutes:** Reviewing and analyzing the solution/correction.
- **30 minutes:** Studying the underlying theoretical concepts to ensure I can master the pattern and solve it on my own later.

Since I'm a beginner, apologies in advance for the messy code I'm about to write to solve this first problem! 

## The exercise

> 
> 
> 
> Given a string `s` containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.
> 
> An input string is valid if:
> 
> 1. Open brackets must be closed by the same type of brackets.
> 2. Open brackets must be closed in the correct order.
> 3. Every close bracket has a corresponding open bracket of the same type.
> 
> **Example 1:**
> 
> **Input:** s = "()"
> 
> **Output:** true
> 
> **Example 2:**
> 
> **Input:** s = "()[]{}"
> 
> **Output:** true
> 
> **Example 3:**
> 
> **Input:** s = "(]"
> 
> **Output:** false
> 

## My idea

My initial idea was to store bracket pairs using a mapping and loop through the string `s`. Every time I encountered a bracket, I wanted to check if its corresponding pair existed in the mapping. If it did, I'd return true, otherwise false. I spent 15 minutes mapping this out, but I realized early on that this approach wouldn't fulfill all the requirements.

## My solution

```python
class Solution:
    def isValid(self, s: str) -> bool:
        mapping = { ")":"(", "}":"{", "]":"["           for c in brackets:
              }
      stack = []
          for i in s:
              for j in mapping:
                  if s[i] == mapping[j]:
                      for k in s[-1]:
                          if s[k] == mapping[j][0] or s[k] == mapping[j][1]:
                              return True
      return False
```

## One possible solution

```python
class Solution:
    def isValid(self, s: str) -> bool:
        stack = []
        mapping = {")":"(", "}":"{", "]":"["}

		        for char in s:
		            if char in mapping.values():
		                stack.append(char)
		            elif char in mapping.keys():
		                if not stack or mapping[char] != stack.pop():
		                    return False
		        
		        return not stack
```

## Learning

### What is a stack in python ?

A stack in python is a simple data structure that follows the LIFO rule: last in, first out. 

A stack only allows operations at **one end**, called the top. 

- push: add an element
- pop: remove the most recent element
- peek: look at the top element
- is_empty: check if the stack has items

```python
stack = []
stack.append(10)   # push
stack.append(20)
stack.pop()        # pop → returns 20
```

append() and pop() are $O(1)$ operations.  

### What is a hash map?

A **hash map** is a data structure that lets you store and retrieve values *extremely fast* using **key → value** pairs. In Python, the built‑in type that implements a hash map is the **dictionary (**`dict`**)**. A hash map uses a **hash function** to convert a key into a position (index) in memory.

This gives $O(1)$ average‑time operations for:

- inserting
- deleting
- looking up

## Skills

### Slicing

From a list, how to skip the first element? 

```python
list = [a, b, c, d, e]
for element in list[1:]: #it will start at b, if it was [2:], it would be started at c
	print(element)
```

How to begin from the end? 

```python
list = [a, b, c, d, e]
for element in list[::-1]: 
	print(element) #it will print: e, then d, then c...
```

### Dictionary

what are the differences between `.values(`) and `.keys()`?

`.keys()` returns a view object displaying all the keys. 

```python
my_dict = {'apple': 1, 'banana': 2}
print(my_dict.keys())  # Output: dict_keys(['apple', 'banana'])
```

`.values()` returns a view object displaying all the values. 

```python
my_dict = {'apple': 1, 'banana': 2}
print(my_dict.values())  # Output: dict_values([1, 2])
```

To have both the values and the keys, we can use `.items()`. 

## Sources

1. https://www.youtube.com/watch?v=-CGiyb1Loyc
2. https://neetcode.io/practice/practice/neetcode150
