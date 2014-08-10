---
layout: post-no-feature
title: "the incomprehensible smoothsort"
description: "Exploring the mechanics behind Dijkstra's sort algorithm: how it achieves 'optimal' asymptotic space and time bounds, and whether it is practical."
category: articles
tags: [sample post, readability, test]
---

So you're lost in the desert.

It's been days, probably weeks, maybe months you've been lost here. At some point you decided to record how long you've been sleeping in this sandy abyss. 

With no writing tools at your disposal, and spending each day wandering in search of escape, the only stable way to record your stay's duration is by flipping a set of heavy flat asymmetric rocks, of which there is a long row placed conveniently nearby a small oasis you've been visiting. You use the rocks as bits -- what else? -- and each new arrangement of rocks represents another day on the calendar.

Flipping the rocks at the end of each day is hard work -- and there's hardly any food in the desert, so you've got to conserve energy. In standard binary notation, you might have to flip seven rocks on day 64. You realized that you can represent numbers as large as you want without flipping more than three rocks on any given day, so you start counting like this:

10,    11,    100,    110,    1000,  1010,    1011,   1100,    10000,    10010,    10011,    10100,    10110,    11000,    100000...

In order to prevent bit-flips what you had to do was prevent carrying; to prevent carrying you need to get rid of strings of consecutive 1s, so any two consecutive 1s are promoted to the next digit after being formed. The result is what people back in civilization might call the Leonardo representation; you calculate the number referred to by each representation by adding up the Leonardo numbers corresponding to each 1. 

The Leonardo numbers run 1, 1, 3, 5, 9, ... i, j, i + j + 1, ... and their recurrence relation corresponds to the condensing we used to create the representation. And you can recall, from before you were stuck here, that they were made famous by one Edsger Dijkstra, for some task or other.

# Good ol' heapsort

Heapsort is a generalized class of sorting algorithms, but our concern is the in-place variants. In each case, we pass over an array and give it some partial ordering, then work backwards from the partial ordering to end up with a sorted array. The partial ordering is always an embedded heap structure, any of various trees where each node is greater than its child nodes, or each node is less than its child nodes; the idea is the same in both cases. A hierarchy of this sort is said to have *the heap property*.

[Without loss of generality](http://en.wikipedia.org/wiki/Without_loss_of_generality), we can assume all heaps are max-heaps.

The standard in-place implementation of heapsort uses a binary heap. For demonstration, here I represent a heap-like ordered array by describing the index of the smallest value guaranteed to be larger than the value at a given index. A binary heap in an array looks like this, with 0 as the largest element: each element k is larger than its two children at 2k + 1 and 2k + 2. 

0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | ...
--|----|---|---|---|----|---|---|---
* | 0 | 0 | 1 | 1  | 2 | 2 | 3 | ...

After we proceed across the array organizing it into a max-heap, the next step is to *dequeue* each element in order, starting from the top of the max-heap, and switching it with the end of the array. An array containing a partially dequeued heap looks like this:

0 | 1 | 2 | 3 | 4 | [...] | k    | k+1  | k+2  | [...] | n-2 | n-1 | n
---|---|---|---|---|------|-----|-------|-------|------|-----|-------|---
k | 0 | 0 | 1 | 1 | [...] | k+1 | k+2 | k+ 3 | [...] | n-1 | n     | *

The algorithm thus proceeds as follows: elements are arranged into a binary heap and then repeatedly dequeued until we are left with a sorted array. As each element is dequeued, the heap has to be fixed -- in the heap above, 0 is about to switch with element k-1, and the new element will be repeatedly swapped with the larger of its children until we have a correct heap again. 

Unfortunately, element k-1 might be the *worst* choice to be at the top of the heap; it came from the bottom, and now it most likely has to be swapped all the way back there. It takes O(log n) steps to do this in almost every case.

Heapsort is an attractive candidate for optimization for a couple reasons: for one thing, it's already one of the quickest algorithms out there. On average, it beats every other comparison sort but quicksort  when sorting large arrays, and it's simple enough -- unlike search-tree methods -- to be efficient on small arrays as well.

Plus, the "heap" property is remarkably general; there are only so many ways to split an array across a pivot, but there's no reason a heap needs to be confined to a binary tree. In the example above, we can see that the whole array is still in some sense a heap -- the sorted portion of an array clearly obeys the heap property. 

It's tempting to think that we might be able to do substantially better by choosing a better heap structure; even at a glance, it seems clear that almost anything should do better than an algorithm which always chooses the smallest elements to move to the top of the heap!

In particular we might ask something like this: can we find a way to "quickly" fix the heap structure, in less than O(log n) time? In general the answer is no; combinatorially, as we build the heap -- for most "normal" in-place heaps -- we perform O(n) comparisons, and each comparison gives us O(n) bits of information about the original structure of the array. Since an array of length n has n-factorial possible arrangements, we need O(log n!) = O(n log n) bits to describe the whole structure, so no matter how clever our algorithm is, we can't always fully predict how the heap should look after each dequeue.

Of course this isn't the end of the story: insertion sort and bubble sort both run in O(n) time if the array is close enough to being sorted. Perhaps under this additional assumption, we can do better? A simple goal is to write an algorithm that won't perform any swaps when it's faced with a sorted array.

In order for this to happen, we want the maximum element of the max-heap to be at the *right*, so the heap might look like this when it's dequeueing, if k-1 (@ symbol) is the highest element still on the heap:


[...] | k-3 | k-2 | k-1@ |  k    | k+1 | k+2 | [...] | n-1 | n
-----|------|------|-------|------|------|------|-----|------|----
      |  k-1 | k-1 |   k    | k+1 | k+2 | k+3 | [...] |  n  | *

And we run into a big problem if we try to use our binary heap: we have to rearrange the whole thing when we lose the top element! Even if I, the programmer, only care about arrays that are already in order, the algorithm must still keep track of the whole structure at every step in order to function as a general sorting algorithm. There are some tricks to do this with a binary heap, but none of them are faster than O(log n) for even presorted data.

So if we are to make an adaptive heapsort, we need to define a structure that doesn't change very much when an element is added or removed. Since we want to be able to fix the heap rapidly for partially-sorted data, it also makes sense to think of the structure as being associated with an *index*, either implicit or explicit, where by index we mean a list of locations where we're likely to find the new top of the heap. 

Since the index is related to the structure, it too must not change very much when an element is added or removed. Since the structure needs to be possible at every given heap-size from 1 to n, since we add and remove elements one at a time, it makes sense to think of the index as a representation of the number k, the size of the heap. A good representation doesn't change much between k and k+1.

And now we're back in the desert, flipping rocks.

# Dijkstra heaps of Leonardo heaps

Our goal is to find an "adaptive" heap structure, characterized by an "index", for performing heapsort in such a way that sorted arrays are never desorted and partially sorted arrays are not heavily rearranged.

Maybe the Leonardo representation makes a good index? It is, after all, the shortest such representation that retains the property "only changes a small number of bits between any two consecutive numbers", though proving this is sort of arduous (one can also start the Leonardo-like numbers 1, 2, 4, 7, 12, ... -- this is a little shorter, but more work in practice). 

In particular, we might define a branch as a sequence of nodes which are each the child of the previous node; we want to use as our index a branch corresponding to the Leonardo representation. So in a heap with k elements and Leonardo representation k = L_1 + L_2 + L_3 + ... + L_m (not the first m Leonardo numbers), with L_i > L_i+1, the "end" of the index-branch ought to be at position L(a_1)-1 in the array, and the "root" at position k-1. Our heap looks like this:

[...] |      L_1-1    | [...] |    L_1+L_2-1    | [...] | k-L_m-1 | [...] | k-1
-----|---------------|-----|---------------------|-----|------------|-----|-----
[...] | L_1+L_2-1 | [...] | L_1+L_2+L_3-1 | [...] |    k-1    | [...] |  *

In between the index-branch nodes we still need to preserve the heap property. However, since each gap has width some L_i we have reduced the problem to finding useful heap structures with sizes of the Leonardo numbers. 

It turns out we can just use the Leonardo numbers themselves, by defining the Leonardo heaps:

LeonardoHeap(i) = {

   root_value

   LeonardoHeap(i-1)

   LeonardoHeap(i-2)

}

The nth Leonardo heap is created by adding a root whose children are the (n-1)th and (n-2)th Leonardo heaps. The 0th and 1st Leonardo heaps are just empty nodes. If we stick on the Leonardo heaps at each element in our index, we obtain a complete heap structure of any size.

Does this really work? Can we add another element and easily maintain a Leonardo representation with attached Leonardo heaps? Well, if the representation has two consecutive 1s as the least significant nonzero bits, we can condense those to the next largest Leonardo heap -- the condensation trick of the Leonardo numbers saves us again. If it doesn't, we add a one-element heap, with the added rule that we count it as the "1" heap only if we don't already have a 1-element heap; this way, we'll always be ready to merge it with the "2" heap if that happens to be hanging around. 

When we pop elements off Leonardo heaps, we get two new Leonardo heaps, unless we're working with a one-element heap, in which case we get nothing. Since we always take from the smallest heap -- at the right side of the array -- we're left with exactly the Leonardo representation of k-1 after removing element k. Dequeueing, finally, works in constant time.

So we have, finally, a heap structure that can be maintained in constant time for sorted data. The algorithm was invented by Edsger Dijkstra, so we call this a *Dijkstra heap*. 

# it's called smoothsort because it adapts smoothly to partially ordered data.

Now the algorithm looks like this: we proceed across the array organizing everything in a Dijkstra heap. As we add each element k, we perform heap rotations as necessary to enforce the heap structure; a heap rotation consists of swapping an element with the largest of its children. If adding element k condensed two Leonardo heaps, the structure looks a bit like this:

 [...] |  a  | [...] |  b  | [...] | k-1 | k@ |
------|-----|-----|-----|------|-----|-----|--
  [...] | k? | [...] | k? | [...] |  k? |  *   |

If any of the elements at a, b, k-1 > k, we swap the largest and repeat this process until the heap property is restored. We proceed across the array adding each element until the whole array is in a Dijkstra heap. If the element is larger than most of those that came before it, this operation is close to O(1); at worst it is O(lg (n)). In particular, the larger the element is, if the array is presorted it will be placed in a smaller Leonardo heap, requiring fewer swaps.

This isn't optimal; the asymptotically optimal way to build a heap is to precompute the structure and work from there, guaranteeing O(n) heap construction, but this method is somewhat complex, and not the subject of this section. It's also not necessary for smoothsort's adaptive performance, which is what we're trying to emphasize. It will, of course, be covered below. 

Dequeueing is only slightly harder. When we remove the element k, either it is the root of a Leonardo heap of order 1 -- it has one child, the next on the index-branch -- or it is the root of a Leonardo tree and has three children. In the first case, we do nothing. In the second case, we first rebalance the Dijkstra heap starting from the middle child, which will then be on the index-branch, and then we rebalance the heap from the final child. It looks like this, gaps omitted, where the locations of nodes are derived immediately from the Leonardo representation:

 a | b | c | d |   e  |  f   | k-1 | k@
---|---|---|----|-----|-----|------|---
k  | d | d | k | k-1 | k-1 |   k   | *

 a | b | c | d@ |   e  |  f   | k-1 | k
---|---|---|------|-----|-----|------|--- 
d? | d | d |  *   | k-1 | k-1 |   *   | *

(if d < a, swap with a, etc)

 a | b | c |     d   |   e  |  f   | k-1@ | k
---|---|---|---------|-----|-----|------|--- 
d | d | d | k-1? | k-1 | k-1 |   *   | *

(if k-1 < d, swap with d, etc)

Each of these operations is cheap if the newly indexed nodes are larger than most previous nodes (partially presorted), and O(lg(n)) at worst. 

The result is a sorted array.

# I don't want an essay, I want to know smoothsort.

[The code in these examples is in Lua. Lua uses 1-based arrays, so I apologize in advance. However the language is nearly perfect in every other way, so I suggest learning it anyway.]

Smoothsort is in-place heapsort using a *Dijkstra heap*. The Dijkstra heap is a heap structure that attempts to solve the *dequeueing problem*: when an element is dequeued from the binary max-heap used in standard heapsort, it is usually replaced by one of the smallest elements in the array, meaning that most dequeues are accompanied by swapping the new root all the way to the bottom of the heap. In [an array-embedded] Dijkstra heap, the largest element is at the right, and so it is not swapped anywhere when it is dequeued; furthermore, the structure of Dijkstra heaps guarantees that dequeueing yields another Dijkstra heap, after a few heap rotations.

There is exactly one way a Dijkstra heap may be structured for every integer N, and it corresponds to the Leonardo representation of the number N. The Dijkstra heap is essentially a stack of Leonardo heaps, heaps formed from Leonardo trees, each having size equal to a Leonardo number marked "1" in the Leonardo representation of N. A Leonardo tree is constructed as follows: the first two Leonardo trees are bare leaves, and each Leonardo tree has a root node whose two children are the previous and previous-previous Leonardo trees.

The following Lua code maintains a Leonardo heap with a possibly invalid root, given its order and the array Leonardo[i] of the Leonardo numbers:
 
`function leonardoheap(arr, rootloc, order)`    
`   if order < 2 then return end`    
`   local prev, prevord = rootloc - 1, order-2`    
`   if arr[rootloc - Leonardo[order - 2]] > arr[rootloc - 1] then`    
`      prev, prevord = rootloc - Leonardo[order-2], order-1`    
`   end`    
`   if arr[prev] > arr[rootloc] then`    
`      arr[prev], arr[rootloc] = arr[rootloc], arr[prev]`     
`      return leonardoheap(arr, prev, prevord)`    
`   end`    
`end`    

The Dijkstra heap of size N is a stack of Leonardo heaps with sizes corresponding to the Leonardo numbers in the Leonardo representation of N, so that when it is embedded in the array, it is ordered with the smallest at the right (top) and the largest at the left (bottom), and each heap's root node is larger than the root of the heap to its left, so the Dijkstra heap is in fact a heap. In the case of 23, its Leonardo representation is 101100, and the Dijkstra heap consists of Leonardo heaps of sizes 15, 5 and 3, from left to right, so the roots of these heaps are at positions 14, 19, and 22 (15, 20, and 23 in Lua) in the array. We call the portion of the Dijkstra heap consisting of the roots of Leonardo heaps the *index-branch* because it forms an index of the structure of the heap, though this is a slight misnomer, as these locations are generally computed on-the-fly.

The following Lua code maintains a Dijkstra heap with a possibly-wrong root node, such as might occur after growing or dequeueing, where leorep contains the indices of ones in the Leonardo representation of rootloc in reverse-sorted order, so 23 = {5, 3, 2} and 38 = {6, 4, 2, 1}:

`function dijkstraheap(arr, rootloc, leorep)`    
`   local nextroot = rootloc - Leonardo[leorep[#leorep]]`    
`   if leorep[#leorep] > 1 then`    
`      local largechild = rootloc - 1`    
`      if arr[rootloc-Leonardo[leorep[#leorep]-2]-1] > arr[rootloc-1] then`    
`         largechild = rootloc - Leonardo[leorep[#leorep]-2] - 1`    
`      end`    
`      if arr[largechild] > arr[nextroot] then`    
`         return leonardoheap(arr, rootloc, leorep[#leorep])`    
`      end`    
`   end`    
`   if arr[nextroot] > arr[rootloc] then`    
`      arr[nextroot], arr[rootloc] = arr[rootloc], arr[nextroot]`    
`      table.remove(leorep)`    
`      return dijkstraheap(arr, nextroot, leorep)`    
`   end`    
`end`    

We are now ready to write smoothsort, assuming an iterator that generates Leonardo representations of integers. This is left as an exercise to the reader.

`function smoothsort(arr)`    
`   for n, leorep in leonardo_representations(1, #arr) `    

We simply add each element to the heap:

`      dijkstraheap(arr, n, leorep)`    
`   end`    

Then we dequeue by proceeding backwards across the array:

`   for n, leorep in leonardo_representations(#arr-1, 1) do`    

Each time we remove an element, if it was the root of a Leonardo heap of order greater than 1, its two child heaps become the first two spots on the index-branch of the Dijkstra heap. We rebalance the Dijkstra heap rooted at its *left* child first, and then rebalance the Dijkstra heap at its *right* child. 

`      if leorep[#leorep] = leorep[#leorep-1] + 1 then`    

(If these heaps are consecutive sizes, the removed node was their parent)

`         local topsize = leorep[#leorep]`    
`         leorep[#leorep] = nil`    
`         dijkstraheap(arr, n - Leonardo[temp], leorep)`    
`         leorep[#leorep+1] = topsize`    
`         dijkstraheap(arr, n, leorep)`    
`      end`    
`   end`    
`end`    

We're done! The algorithm runs with O(n lg n) worst case performance but O(n) performance in a presorted array, and, as Dijkstra noted, a smooth transition between the two for input arrays of varying presortedness.

The worst case for smoothsort, incidentally, is an array in reverse order, or nearly-reverse order. A clever implementation might check for this and flip it, allowing adaptiveness for reverse-sorted arrays.

# that's great, sir, but I live in the real world

Smoothsort was not the first nor the last adaptive-and-worst-case-optimal sorting algorithm, you say. Most of those didn't get used because they have terrible real-world performance: they require large space overhead, complicated data structures, or lots of extra variables and structure to keep track of as the algorithm runs so any performance benefits are only recognizable at the extremely high end of array size. Why should smothsort be any different? 

To add insult to injury, your typical Google result for smoothsort links this benchmark:

http://www.altdevblogaday.com/2012/06/15/smoothsort-vs-timsort/

In it an example implementation of smoothsort written to support a blog post is compared against C++'s std::sort. For somewhat obvious reasons std::sort() wins by some orders of magnitude.

Fortunately for us, smoothsort hasn't quite been given a fair shake. The algorithm is fully in-place, uses only one data structure, and makes two passes over the array; one forward, one backward. It doesn't have the cache performance of quicksort, though. I attempted to write a benchmark implementation with some simple optimizations:

https://gist.github.com/scythe/a4a5c7d5e70866bdd5e1

# tl;dr

*In selection sort, we run through the array, find the largest element, and put it at the end, then repeat. This is wasteful: we make a lot of extra comparisons as we find the largest element, and it would be nice if we could keep track of them and use them for later. We make use of this by passing through the array and placing large elements at indices corresponding to sums of distinct Leonardo numbers greater than 2 such that each element at such an index is larger than the three elements with the first offset before it by the least Leonardo number in the sum, and the second after the first by the Leonardo number just smaller than this, and the third after the second by the Leonardo number just smaller than that, and by simply ensuring elements not fitting this pattern are larger than the previous element. By maintaining this structure as we pass up and down the array, it takes a minimal amount of time to find the next highest element for the sort.*
