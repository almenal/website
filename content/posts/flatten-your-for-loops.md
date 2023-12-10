---
title: "Flatten Your for Loops"
date: 2023-12-10T21:35:40Z
draft: true
---


## Intro

Recently I found myself  writing the following code

```python
datasets = [...]
groups = [...]
methods = [...]
for dataset in datasets:
	for group in groups:
		for method in methods:
			do_something(dataset, group, method)
```

Triply nested loop!
Sorry I should’ve added a trigger warning for the 
[never nesters](https://www.youtube.com/watch?v=CFRhGnuXG-4&t=79s&pp=ygUMbmV2ZXIgbmVzdGVy).

I wouldn’t go as far as to label myself a “never-nester”,
but still I felt an itch in the inside of my brain.
If you also felt it, here’s a quick tip from me to you: use `itertools`!

With `itertools.product` we can flatten our for loop into a single level of indentation, like this:

```python
combinations = itertools.product(datasets, groups, methods)
for (dataset, group, method) in combinations:
	do_something(dataset, group, method)
```

With that simple trick, we went from several lines and levels of indentation 
to a flatter structure that fits better in our minds.
But why should you do that? Or rather, when should or shouldn’t you flatten for loops?

## Reason 1: Easier to follow along

Every level of indentation opens a fork in your mind.

One of Python’s distinguishing features is making indentation a core part of the syntax.
Loops, if statements, functions, classes… are all delimited by indentation.
Every new level should correspond to a “thing”,
and your reasoning about the “thing” should happen within that block.
As soon as you go back, the block, or “thing” is over.
Python links the horizontal structure of the code and the flow of the program. 

Triple nested loops like the one above is a single child wearing three trenchcoats.
They are not three things, they are just one:
applying a function to every combination of possible arguments.
It doesn’t matter the order in which the arguments are iterated, as long as we do all of them.
Every level of indentation doesn’t correspond to a new “thing”,
so your brain is wasting energy keeping track of two extra “things”.

By flattening the for loop and iterating over `combinations`,
we simplify this structure and immediately know that the code inside 
the for loop is doing one thing only. 

`itertools` has the added perk that is scales well in terms of memory efficiency.
`combinations` is not a *list*, but a *generator.*
It doesn’t store all combinations in memory,
but generates them dynamically at each iteration.
So whether your lists are tiny or massive, you won’t suffer from combinatorial explosion 
(at least not at the for loop level).

>  Shoutout to Julia for letting you define these sort of nested for loops in one line.
[link to julia docs](https://docs.julialang.org/en/v1/manual/control-flow/#man-loops)

## Reason 2: progress bars

With deeply nested loops, progress bars will give you less granular updates.
If your `datasets` list has 3 elements, but `groups` has 10 and `methods` has 5,
then you’re getting an update every 50 iterations.
If you’re trying to spot a bottleneck 
(e.g. maybe one method takes twice as long as the rest),
you’d be missing out on this information.

## Reason 3: parallelization

Building on the bottleneck issue we mentioned above:
what if the program took a very long time to run and you wanted to speed it up 
using a parallelisation strategy like `multiprocessing`?
Which loop level should you parallelize? The innermost one?
The outermost one? Whatever you choose, you would not take full advantage of parallelisation,
because you’d be grouping iterations together that could be split up into independent processes.

## Caveat 1: caching

There is one case where we might want to keep the for loops separate.
If there is some object that is expensive to compute and that can be computed between loops,
then it may make sense to do that computation once and re-use it within the inner loop. 

```python
inner_combis = itertools.product(groups, methods)
for dataset in datasets:
	expensive_object = generate_expensive_object(dataset)
	for group, method in inner_combis:
		do_something(expensive_object, group, method)
```

## Caveat 2: maybe it is a `group_by` in a trenchcoat…?

This is more of a specific use case, but one I come across often.
If you’re handling `pandas.DataFrame` s and you’re iterating over groups with a for loop,
maybe what you need is a `group_by` operation, and not a for loop.

