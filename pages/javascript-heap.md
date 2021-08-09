---
permalink: javascript-heap
mermaid: true
title: JavaScript Heap
---

If you've landed here in the heat of an interview here is a Javascript heap implementation:

```js
class MinHeap {
  constructor() {
    this.data = [];
  }

  peak() {
    return this.data[0];
  }

  push(value) {
    this.data.push(value);

    let i = this.data.length - 1;
    while (i > 0) {
      const parentIndex = Math.ceil((i / 2) - 1);
      if (this.data[i] < this.data[parentIndex]) {
        this.swap(i, parentIndex);
        i = parentIndex;
      } else {
        break;
      }
    }
  }

  pop() {
    // 1 or no remaining items is a special case
    if (this.data.length < 2) {
      return this.data.pop();
    }

    const min = this.data[0];
    this.data[0] = this.data.pop();

    let i = 0;
    while (true) {
      const [leftIndex, rightIndex] = [(i * 2) + 1, (i * 2) + 2];
      const leftValue = this.data[leftIndex] ?? Infinity;
      const rightValue = this.data[rightIndex] ?? Infinity;

      // If both children are larger than the candidate, we're done.
      if (leftValue > this.data[i] && rightValue > this.data[i]) {
        break;
      }

      // Otherwise pick the index of the smallest value
      const smallestIndex = leftValue < rightValue ? leftIndex : rightIndex;

      this.swap(i, smallestIndex);
      i = smallestIndex;
    }

    return min;
  };

  swap(i1, i2) {
    const val1 = this.data[i1];
    this.data[i1] = this.data[i2];
    this.data[i2] = val1;
  }
}
```

And if you just need something to quickly copy-and-paste:

```js
class MinHeap{constructor(){this.data=[]}peak(){return this.data[0]}push(t){this.data.push(t);let a=this.data.length-1;for(;a>0;){const t=Math.floor((a-1)/2);if(!(this.data[a]<this.data[t]))break;this.swap(a,t),a=t}}pop(){if(this.data.length<2)return this.data.pop();const t=this.data[0];this.data[0]=this.data.pop();let a=0;for(;;){const[t,s]=[2*a+1,2*a+2],h=this.data[t]??1/0,i=this.data[s]??1/0;if(h>this.data[a]&&i>this.data[a])break;const d=h<i?t:s;this.swap(a,d),a=d}return t}swap(t,a){const s=this.data[t];this.data[t]=this.data[a],this.data[a]=s}}
```

## Why Use a Heap?

A heap can serve as a sorted stack, prioritizing the smallest value (minimum
heap) or the largest (maximum heap). From here on we'll focus on minimum heaps,
but the same principles apply to maximum heaps. And, although there are many
documented implementations for heaps, one of the most practical and popular is
based on a binary tree. This is the implementation you'll see, for instance, in
[Java's PriorityQueue].

Getting the minimum item from a heap is performed in constant time O(1) while
adding a new item to the heap or popping an item from the heap is performed in
O(log n).

In a simple case, you may be asked in an interview to create a data structure
that can return the minimum item in O(1) time. More practically, a heap can
serve as a queue where items with the higher priority are handled before items
with lower priority.

Here's how our heap will behave:

```js
const heap = new MinHeap();
heap.push(2);
heap.push(1);
heap.push(3);

heap.pop(); // 1
heap.pop(); // 2
heap.pop(); // 3
```

Array-Backed Heap
-----------------

Although heaps can be implemented with a tree of node objects, they are more
often implemented on top of arrays because:
* Arrays can represent a binary tree with minimal overhead
* This binary representation naturally stays balanced when adding and removing items

This is a bit of a trick, but with a little math we can turn our tree into an
indexed list of items.

Given a tree with three nodes.

<div class="mermaid">
graph TD
  1 --> 2
  1 --> 3
</div>

We can represent it with an array of three items.

```
[1, 2, 3]
```

When we push a new items onto the array, it fills out the tree breadthwise from
left to right.

<div class="mermaid">
graph TD
  1 --> 2
  1 --> 3
  2 --> 4
  2 --> 5
  class 2 emphasis;
  class 4 emphasis;
  class 5 emphasis;
</div>

```
[1, 2, 3, 4, 5]
    ↑     ↑  ↑
```

<div class="mermaid">
graph TD
  1 --> 2
  1 --> 3
  2 --> 4
  2 --> 5
  3 --> 6
  3 --> 7
  class 3 emphasis;
  class 6 emphasis;
  class 7 emphasis;
</div>

```
[1, 2, 3, 4, 5, 6, 7]
       ↑        ↑  ↑
```

In our implementation we're going to expressed this relationship with the
following equations:

```js
// Given the index of a child node, get its parent
const parentIndex = Math.ceil(i / 2) - 1

// Given a parent node index, get its left child
const leftIndex = (i * 2) + 1

// Given a parent node index, get its right child
const rightIndex = (i * 2) + 2
```

Implementation Outline
----------------------

Let's start with an implementation that isn't a heap at all. Let's begin by
creating a stack backed by an array.

```js
class MinHeap {
  constructor() {
    this.data = [];
  }

  peek() {
    return this.data[0];
  }

  push(value) {
    this.data.push(value);
  }

  pop() {
    return this.data.pop();
  }
}
```

Going back to our example test case, we see that this implementation just
returns items based on insertion order:

```js
const heap = new MinHeap();
heap.push(2);
heap.push(1);
heap.push(3);

heap.pop(); // 3
heap.pop(); // 1
heap.pop(); // 2
```

Implementing Push
-----------------

Now we'll focus on enhancing our `push` method so that it maintains the
properties of the minimum heap. Whenever we finish an operation, each tree node
should be smaller than its two children. This means that the root node of the
tree (position `0` in our array) should be the smallest item in the heap.

Whenever a new item is added, it likely needs to be repositioned. It might even
be a new minimum that needs to make it to position `0` in the array. After
adding a new item, if its parent is smaller than the new item, we'll swap the
new item with the parent. We keep repeating that operation until we've reached
index `0` (new minimum!) or until the parent is smaller than the new item.

```js
push(value) {
  this.data.push(value);

  // i starts at the last index; the index of the item we just pushed.
  let i = this.data.length - 1;

  // If we get to i === 0, then this is the new heap minimum and
  // we can stop.
  while (i > 0) {
    const parentIndex = Math.ceil(i / 2) - 1;

    // If the current value isn't smaller than the parent value,
    // then stop.
    if (this.data[i] >= this.data[parentIndex]) {
      break;
    }

    // If the parent is larger than the current value then we need to
    // swap the values and continue.
    this.swap(i, parentIndex);
    i = parentIndex;
  }
}
```

I chose to extract `swap` to a method because I don't want to see this bit of
tricky code with the rest of the algorithm. The implementation uses a temporary
variable to swap two values in an array:

```js
swap(i1, i2) {
  const val1 = this.data[i1];
  this.data[i1] = this.data[i2];
  this.data[i2] = val1;
}
```

Let's run through an example of the `push()` method in action. Given this array:

```js
[2, 3, 4]
```

When we push 1 we get:

```js
[2, 3, 4, 1]
```

1 is less than the parent index value so we swap
1 with its parent:

```js
// parent index = Math.ceil(3 / 2) - 1 = 1
// parent value = 3
[2, 1, 4, 3]
```

And again, 1 is less than its parent index value so we swap
1 with its parent again.

```js
// parent index = Math.ceil(1 / 2) - 1 = 0
// parent value = 2
[1, 2, 4, 3]
```

At this point the candidate's index is 0 so we stop.

Implementing Pop
----------------

That's more than half of our heap implementation out of the way! Now we just
need to maintain the minimum heap properties in our `pop()` method and we're
done.

Let's start with the easiest situation. If we just have 1 or 0 items in the array
we can just return that value of `data.pop()`—no other work to do.

```js
pop() {
  // 1 or no remaining items is a special case
  if (this.data.length < 2) {
    return this.data.pop();
  }
};
```

With that out of the way, we'll store the minimum value to return at the end
of the method.

```js
pop() {
  // ...
  const min = this.data[0];
};
```

This operation is supposed to remove the minimum from the heap, so we need some
value to put in its place. Although it's unlikely to be the new minimum, we
choose to `pop()` the last item from the array because this shortens the array
without disturbing the qualities of our heap tree. If we choose to `shift()`
earlier to remove the minimum or `splice()` an item from the middle, our array
representation of a tree (where every parent node is smaller than its children)
would be destroyed.

```js
pop() {
  // ...
  // Set a new min candidate
  this.data[0] = this.data.pop();
};
```

With our new minimum candidate at the root of the tree, we'll examine the value
and see if it should "sink" in the tree. If the value is larger than its left or
right child node, then we should swap this value with a child.

```js
pop() {
  // ...
  let i = 0;
  while (true) {
    const [leftIndex, rightIndex] = [(i * 2) + 1, (i * 2) + 2];

    // I'm choosing to set undefined values to Infinity so that they're
    // always larger than the minimum candidate and won't get picked.
    const leftValue = this.data[leftIndex] ?? Infinity;
    const rightValue = this.data[rightIndex] ?? Infinity;

    // If both children are larger than the candidate, we're done.
    if (leftValue > this.data[i] && rightValue > this.data[i]) {
      break;
    }

    // Otherwise pick the index of the smallest value
    const smallestIndex = leftValue < rightValue ? leftIndex : rightIndex;

    this.swap(i, smallestIndex);

    // Continue to evaluate the candidate as it sinks through the tree
    i = smallestIndex;
  }
};
```

And finally we need to return that minimum value we held onto:

```js
pop() {
  // ...
  return min;
};
```

Let's consider an example showing how `pop()` will operate. Given our array
from earlier:

```js
[1, 2, 4, 3]
```

We remove 1 and replace it with the last, popped item.

```js
[3, 2, 4]
```

At this point we determine whether the children values (2 or 4) are
smaller than 3. In this cases 2 is smaller so we swap those values.

```js
[2, 3, 4]
```

Now when we look for the 2 children of 3, they're both undefined so
we're finished.

[Java's PriorityQueue]: (http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/util/PriorityQueue.java)
