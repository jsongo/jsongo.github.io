---
layout: post_item
status: publish
published: true
title: "binary-tree(python)"
author: jsongo
date: '2015-06-05 13:17:45 +0800'
categories: [python, theory]
tags: [python, masterpiece]
comments: []
---
Binary tree written in python
{% highlight python %}
# -*- coding: utf-8 -*-

class Node:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None
        self.parent = None

    def add(self, value): # using recursion
        if value > self.value:
            if self.right:
                return self.right.add(value)
            else:
                self.right = Node(value)
                self.right.parent = self
                return self.right
        elif value == self.value:
            return self
        else:
            if self.left:
                return self.left.add(value)
            else:
                self.left = Node(value)
                self.left.parent = self
                return self.left

class Tree:
    def __init__(self):
        self.root = None # a node

    def add(self, value):
        if not self.root:
            self.root = Node(value)
            return self.root
        else:
            return self.root.add(value)

    def contains(self, value):
        node = self.root
        while node:
            if value > node.value:
                node = node.right
            elif value == node.value:
                return node
            else:
                node = node.left
        return False

    def remove(self, value):
        node = self.contains(value)
        if not node:
            return None
        else:
            if not node.right:
                node.parent.left = node.left
            elif not node.left:
                node.parent.right = node.right
            else: # the node has two child
                largest_at_left = 0
                smallest_at_right = 0
                ori_node = node
                # find the largest in the left or the smallest in the right
                node = ori_node.left
                while node.right:
                    node = node.right
                largest_at_left = node
                node = ori_node.right
                while node.left:
                    node = node.left
                smallest_at_right = node
                # which is closer to the removing one
                if value - largest_at_left.value > smallest_at_right.value - value:
                    node = smallest_at_right
                else:
                    node = largest_at_left
                node.parent = None
                if ori_node.parent.left.value == ori_node.value:
                    ori_node.parent.left = node
                else:
                    ori_node.parent.right = node
                node.left = ori_node.left
                node.right = ori_node.right

{% endhighlight %}
