two-phase-handling
------------------

- Exception catching is handled in two phases: lookup and cleanup (or _UA_SEARCH_PHASE and _UA_CLEANUP_PHASE).
- Let's go again over our exception throwing and catching recipe:

- __cxa_throw/__cxa_allocate_exception will create an exception and forward it to a lower-level unwind library by calling _Unwind_RaiseException
- Unwind will use CFI to know which functions are on the stack (ie to know how to start the stack  unwinding)
- Each function has have an LSDA (language specific data area) part, added into something calle ".gcc_except_table"
- Unwind will try to locate a landing pad for the exception:
  1. Unwind will call the personality function with the action _UA_SEARCH_PHASE and a context pointing to the current stack frame.
  2. The personality function will check if the current stack frame can handle the exception being thrown by analyzing the LSDA.
  3. If the exception can be handled it will return _URC_HANDLER_FOUND.
  4. If the exception can not be handled it will return _URC_CONTINUE_UNWIND and Unwind will then try the next stack frame.
  5. If no landing pad was found, the default exception handler will be called (normally std::terminate).

If a landing pad was found:

- Unwind will iterate the stack again, calling the personality function with the action _UA_CLEANUP_PHASE.
- The personality function will check if it can handle the current exception again:
- If this frame can't handle the exception it will then run a cleanup function described by the LSDA and tell Unwind to continue with the next frame (this is actually a very important step: the cleanup function will run the destructor of all the objects allocated in this stack frame!)
- If this frame can handle the exception, don't run any cleanup code: tell Unwind we want to resume execution on this landing pad.
- There are two important bits of information to note here:
    1. Running a two-phase exception handling procedure means that in case no handler was found then the default exception handler can get the original exception's stack trace (if we were to unwind the stack as we go it would get no stack trace, or we would need to keep a copy of it somehow!).
    2. Running a _UA_CLEANUP_PHASE and calling a second time each frame, even though we already know the frame that will handle the exception, is also really important: the personality function will take this chance to run all the destructors for objects built on this scope. It is what makes RAII an exception safe idiom!
