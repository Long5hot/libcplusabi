LSDA
----

- LSDA on the other hand means language specific data area,
  and it will be used by the personality function to know which exceptions can be handled by this function.

- compiler writes something called LSDA, language specific data area,
  to know which exceptions can a method handle.

- Each function will have an LSDA (language specific data area) part,
  added into something called ".gcc_except_table"
