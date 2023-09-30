# New syntax `Trailing Method` 


## abstract

This pep proposes a new syntax named `trailing compound statement` for instantiating 
complex objects and organized setting attributes.

`Trailing compound statement`, or namely `trailing block`, is an indent code block as well as 
variable scope right after an object is instantiated. The local variables within such 
block are caught and pass to the dunder method `__trailing__` of the forward object.

`Trailing block` syntax may in the first glance seems the `with` statement without `with` keyword itself. 
However, the purpose and what focus on are significantly different. `Trailing block` has nothing to 
do with ***defer*** usage. It is designed to handling complex works listed below, with a clear syntax 
and nice scope arranging.  


## motivation 