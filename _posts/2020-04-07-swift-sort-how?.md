---
title:  "Understanding Swift's Sort: Timsort"
category: ios
date: 2020-04-07
---

Recently, I participated in [Google's Codejam 2020](https://codingcompetitions.withgoogle.com/codejam). It was a qualification round with four different questions to solve, and I tried to use Swift! And as I expected, the efficiency of Swift was horrible, and even with a same algorithm, C++ passed tests that Swift failed on. 

That got me thinking, how does swift sort things? We generally do something like 
```swift
sort({ $0.value < $1 })
``` 
to sort our collections, but other than the sort block, we never really consider which algorithm we use. 

Bubble sort? Selection sort? Quicksort? Let's take a look.

## Swift's sorting algorithm
Previously, Swift was known to use [introsort](https://en.wikipedia.org/wiki/Introsort), which is a hybrid sorting algorithm that switches between quicksort, heapsort, and insertion sort to provide the best performance for corresponding recursion depth level. 

But in [late 2018](https://github.com/apple/swift/pull/19717), the sort algorithm was changed to a modified timsort to provide a more stable, adaptive sorting performance. It "merges using a temporary buffer... performs straight merges instead of adopting timsort's galloping strategy." 

Okay, that's a lot of complex words...

## Timsort

Tim sort is a sorting algorithm introduced by Tim Peters in 2002. It uses insertion sort and merge sort, and has best time complexity of O(n), and average and worst time complexity of O(n log n). It is stable because it uses two stable algorithms, and has overcome the limitation of other O(n log n) sorting algorithms by using less memory. Because it uses different algorithms for given inputs, it also has the advantage of being an adaptive algorithm. 

Timsort uses the idea of dividing the list into separate pieces, sort each piece by using Insertion sort and merging them afterwards. 

Because dividing the input by `2^x` pieces and sorting them using Insertion sort can reduce `x` amount of work by typical Merge sort, it can reduce the time complexity of Merge sort from `Cm * n log n` to `Cm * n(log n - x) + a`. In order to further improve the algorithm, timsort uses more optimization to maximize the value of `x` and reduce the value of `a`.

### Run
The best way to reduce the value of `x` is to sort the list of 2^x elements with Insertion sort, and then using Merge sort, effectively reducing the number of merges. 

Because many real-world data have incrementing or decrementing sequence in the list, `Run` finds and groups the sub-array with ascending or strictly descending elements. Now, the array is divided into units of `minrun`, which is `2^x`. Because merge sort becomes more effective when there is 2^x elements to merge, the number of run should be a power of 2.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200407/timsort-merge-run.png" alt="">

Each of these `run`s will be sorted using Insertion sort, leaving us with number of groups of sorted sub-arrays.

Timsort uses Binary Insertion sort, which assumes that all elements prior to current index are sorted. This saves the algorithm from additioanlly comparing the value to previous elements, reducing the time of inserting the element and shirting other elements. 

### Merge

After forming each `run`, making sure that they are bigger than `minrum`, and ensuring that they are in ascending order, it's time to merge them effectively. Each `run` is added to the stack, but merging before stack them. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200407/timsort-merge-xyz.png" alt="">

In timsort, each `run` may have different length, making typical merge sort really hard. So, it performs merge until a specific requirement is met.
```
A > B + C
B > C
```
Here, the upper items in the stack are **always** bigger than the elements lower in the stack. When this is achieved, each item will be near another item with similar size. 

### Run Merge
With `run A`(smaller) and `run B`(larger) to merge, we first choose `run A` and create a temporary array with the same length. We then copy the entire elements of `run A` into the temporary array. 

Then, we compare each item from `run B` with the temporary values, and fill the empty `run A` array with the comapred value. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200407/timsort-merge-temporary.png" alt="">

This will allow us to ensure that the array is merged in order, very efficiently. 

Another optimization used by tim sort is finding sequence where merging is not needed. 

In below image, for example, the [3,5,8] and [13,15,18,21] sequence is already merged, and only the middle four elements have to be merged. By merging only these four elements, we can save time by finding such unneeded sequencce by binary search and ignoring them. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200407/timsort-merge-binary-search.png" alt="">
This may actually worsen the algorithm as it needs to spend some time binary searching the array, but because it can greatly improve the algorithm with real-world data, it is still used in timsort. 

### Galloping

Lastly, we'll look at something called `Galloping`. What's galloping anyways?
> Gallop [noun]: the fastest pace of a horse or other quadruped, with all the feet off the ground together in each stride.

And it exactly means just that in timsort too. 

Because merge sort compares items 1 by 1, it can often be better to merge elements in chunk if we can ensure that they are consecutively sorted. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200407/timsort-merge-galloping.png" alt="">

Timsort keeps the count of consecutive elements selected from a run. If the elements of Y are consecutively smaller than the elements in the temporary array and reaches the *minimum galloping threshold*, timsort switches to the **`galloping mode`**, moving the chunk as a whole. Timsort compares the elements in galloping mode by comparing in `2^x`. When the elements stop being consecutively merged from a single `run`, timsort returns to its normal `one pair at a time mode`. 

---
With that, timsort uses merge sort, but divides the array into separate `run`s and optimize the number of merging performed. 

This allows it to be very fast, stable, and efficient in a real-world generic data, which is why it's used as a standard sorting algorithm in many different systems. 

## Now, back to Swift

Previously, we said that Swift uses a modified timsort: `straight merges instead of adopting timsort's galloping strategy`. 

This means that Swift does not uses the `galloping` strategy described above, not merging elements by chunks when feasible. However, it still outperforms Introsort in many scenarios, even more so with real-world data!

This is really complex, but it's still rewarding to know the innerworkings of things we use so frequently. 

You can learn more about Swift's Sort implementation in its [open sourced library](https://github.com/apple/swift/blob/master/stdlib/public/core/Sort.swift).  