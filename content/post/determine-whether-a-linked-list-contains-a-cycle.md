---
title: Determine Whether a Linked List Contains a Cycle
date: 2023-02-24T00:01:05.797Z
draft: false
---

The common solution is to use two pointers, one fast and one slow. I recently remembered with amusement that in October of 2018, I came up with and submitted to LeetCode a different algorithm:

1. Hold on to the head of the linked list.
2. Traverse the linked list, reversing it as you go.
3. Eventually the current node will have no next node. Compare the current node with the head node. If they are one and the same, then the linked list contains a cycle.
4. If the linked list needs to be restored to its original state, reverse the linked list again.