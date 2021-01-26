---
layout: post
title:  "Algorithms in Go: Merge Intervals"
date:   2016-01-01 13:05:00
categories: programming
tags: algorithms go
---
This is the third part of a series covering the implementation of algorithms in Go. In this article, we discuss the Merge Intervals algorithm. Usually, when you start learning algorithms you have to deal with some problems like finding the least common denominator or finding the next Fibonacci number. While these are indeed important problems, it is not something that we solve every day. Actually, in the vast majority of cases, we only solve such kinds of algorithms when we learn how to solve the algorithms. What I like about the Merge Interval algorithm is that we apply it in our everyday life, usually without even noticing that we are solving an algorithmic problem.

Let's say that we need to organize a meeting for our team. We have three colleagues Jay, May, and Ray and their time schedule look as follows (a colored line represents an occupied timeslot):
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/ac1/2ca/b6c/ac12cab6cb49264e89d3213d96fd210a.jpg" %}

Which timeslot should we pick up for a new meeting? One could look at the picture, find a gap in Ray's schedule, and check whether the others a gap there as well.
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/939/7ac/b13/9397acb13fcb9762a222329815923e21.jpg" %}

How can we implement this logic? Most straightforwardly, we can iterate through every minute of the day and check whether someone is having a meeting during that time. If none of the colleagues are occupied at that time, we find an available minute.

How can we simplify this approach? Instead of checking all employees for every minute, we can merge their schedules and find the available slots in the resulting schedule.
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/d7a/7f3/ff1/d7a7f3ff1dca11ae4bae14f3aeb29df8.jpg" %}
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/842/4ef/fb1/8424effb1863d820b5384fb982c2afbc.jpg" %}
{% include image.html src="https://habrastorage.org/getpro/habr/upload_files/774/39d/d59/77439dd598a857b617764b732acc7ace.jpg" %}

After the merge, we can iterate through the array of the merged meetings and find the gaps.

How can we implement the algorithm above? Let's create type `Slot` that represents a time slot in the schedule; for simplicity, we use integers to denote the start and the end of the time slot instead of `time.Time` type.

{% highlight go %}
type Slot struct {
  start int
  end int
}
{% endhighlight %}

For each of the employees will have a sorted array of Slots (staring from the earliest time slot) that represent occupied time slots in their schedule. Our function will take an array of Slot arrays that represents the schedule for the whole team as an input parameter.

{% highlight go %}
[][]Slot {
  \{\{9, 10\}\},         // occupied time slots for John
  \{\{1, 3\}, {5, 6\}\},  // occupied time slots for James
  \{\{2, 3\}, {6, 8\}\},  // ...
  \{\{4, 6\}
}
{% endhighlight %}