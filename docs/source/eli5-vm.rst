BEAM VM ELI5
============

BEAM got its name from Bogdan/Björn Erlang Abstract machine. This is register
VM, but it also has a stack.

Registers ELI5
--------------

Being a register VM means that stack is not used to pass arguments
between calls, but registers are. They are not same registers as one would
expect to see in a CPU architecture, but rather just an array of
:ref:`Term <def-term>` values. There are 1024 registers but functions of such
arity are rare, so usually VM only uses first 3 to 10 registers.

For example, when a call to a function of arity 3 is made: registers x0, x1,
and x2 are set to function arguments. If we ever need to make a recursive
call, we will have to overwrite registers and lose the original values in
them. In this situation, we will need to save old values on the stack. There
is no dynamic stack allocation other than stack frame, determined by the
compiler.


Bytecodes and VM Loop ELI5
---------------------------

During BEAM code loading, some combinations of opcodes are replaced with a
faster opcode. This is optimisation trick called *superinstruction*.
For each opcode, there is a piece of C code in ``beam_emu.c`` which has a
C label. An array of C labels is stored at the end of the same VM loop routine
and is used as the lookup table.

After the loading opcodes are replaced with such label addresses, followed by
arguments. For example, for opcode #1, an element with index 1 from labels
array is placed in the code memory.

This way it is easy to jump to a location in C code which handles next opcode.
Just read a ``void*`` pointer and do a ``goto *p``. This feature is an
extension to C and C++ compilers. This type of VM loop is called
*direct-threaded dispatch* virtual machine loop.

Other types of VM loops are:

*   *switch dispatch* VM (this one is used by BEAM VM source if the C compiler
    refuses to support labels extension. It is also 20 to 30% slower than direct
    threading;
*   *direct call threading*
*   and a few more exotic
    http://www.cs.toronto.edu/~matz/dissertation/matzDissertation-latex2html/node6.html