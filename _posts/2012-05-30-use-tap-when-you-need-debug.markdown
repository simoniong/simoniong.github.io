---
layout: post
title: "use tap when you need debug"
date: 2012-05-30 21:34
comments: true
categories: [ruby, debug]
---

When you try to debug in a chain methods like following example:
``` ruby
[1,2,3,4,5].collect { |x| x + 1 }.inject { |sum, x| sum + x }
```

The typical way is to break the chain block, then insert some shitty print code in the middle block, then introduce new temp variales so that it can keep chaining, one typical solution is demonstrated below:

``` ruby
temp = [1,2,3,4,5].collect do |x| 
           p x
           x + 1
         end
temp.inject { |sum, x| sum + x }
```

This always is not a good solution. When we break the middle block, it have chance to pollute x variable, and introduce new bug if we forget to change back after debug.

Actually, we can use object#tap method to make this debug more elegant. Typically usage is to inject a tap block in the debuging object, and print the object out in the tap block. In the example above, this debuging object is x of block collect. 

``` ruby
[1,2,3,4,5].collect { |x| x.tap { |o| p o } + 1 }.inject { |sum, x| sum + x }
1
2
3
4
5
=> 20
#
```

tap method can inject into any object, so that we can use this method to avoid breaking out the block. and one more thing: tapped variable will not be pollute in block, so that we can avoid accidently pollute variable we are interesting with and prevent to introduce new bug during debuging.
