---
layout: post
title: Guard Your Bases
---

Sometimes you must "guard" the execution of a block of code with a condition
to prevent bad things...

{% highlight ruby %}
def process
  puts "Processing..."
end

process if process_enabled?

# Meanwhile, in another code block
process if process_enabled?
{% endhighlight %}

In this example the `process_enabled?` method guards the execution of `process`
in the case that processing is disabled. Checking for this condition really
isn't a concern of the caller. Not only does it add responsibility to the
caller, it also introduces duplication, and creates a great opporunity for a
bug to be introduced.

The solution is to move this "guard" condition as close the the "base"
operation as possible. Doing so provides a number of wins:

1. The responsibility of checking the condition more appropriately belongs to
   the executor of the ...
2. The guard effectively prevents "processing" for every attempt rather than
   only those which are explicitly guarded.
3. The potential removal of this guard requires a single change at the point
   of interest rather than throughout the codebase.
