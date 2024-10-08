__cxa_allocate_exception
------------------------

__cxa_allocate_exception receives a size_t and allocates enough memory to hold the exception being thrown.
There is more to this that what you would expect: when an exception is being thrown some magic will be happening 
with the stack, so allocating stuff here is not a good idea. Allocating memory on the heap might also not be a good idea,
though, because we might have to throw if we're out of memory. A static allocation is also not a good idea,
since we need this to be thread safe (otherwise two throwing threads at the same time would equal disaster).
Given these constraints, most implementations seem to allocate memory on a local thread storage (heap)
but resort to an emergency storage (presumably static) if out of memory.
We, of course, don't want to worry about the ugly details so we can just have a static buffer if we want to.
