

This document needs to be filled out, but for now I need to note recursion.

Recursion in the VM happens when a VM function calls itsself. When this happens the local stack is saved, and a new one 
is spawned for the new call into the function. If data is to be preserved between recursive calls the global stack or registers must be used.
This is to ensure that the local stack from the originating (first) recursive call is maintained.
The jump stack for (@ret and @jmp) are also saved per-function to ensure jumps aren't trampled


Recursion using pcall is still up for debate, I dont see much use in it so Its being put on a back burner right now.

