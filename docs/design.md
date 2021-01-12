# Design

This page documents the design.

## Interpreter agnostic

The internal of the actor-based runtime pallet is designed to be
interpreter agnostic. With proper implementation, it should allow you
to plug in WebAssembly, EVM or RISC-V.

However, as now, for simplicity, we experiement the actor-based model
using a subset of EVM called `evm-core`. This is EVM with all
Ethereum-specific features removed.

The reason why we haven't put in WebAssembly for the initial
experiment is due to the limitations of APIs we have -- actor-based
model will likely require saving the full contract running state
(including the memory and the stack), but unfortunately this is not
something we can yet do in Substrate.

## Actor lifecycle

An actor is a single unit of contract. Actors are never designed to
stop its execution. This means that it will keep its memory and stack
data forever. When an actor ends, either due to an explict exit or
normal execution stop, then it is self-destructed and removed from the
system (together with all the data).

The design encourages small and single-purpose smart contract "units"
and multiple "units" communicating and working with each other, which
can hopefully reduce the risk of error.

The design also reduce extra constructs an actor would ever need. To
write an actor, developers are presented with the familiar workflow of
writing a simple `main` function, with only several additional normal
system API calls for doing things like sending messages.

## Data in an actor

Unlike `pallet-contracts`, where there is an explict concept of
contract storage, in an actor, there are only the code, the heap
(memory), and the stack. The interpreter for the actor-based framework
should limit the size of code and stack so that they can be easily
loaded with the contract metadata. The memory is loaded only when
needed and is charged by pages.

## Actor identifier

The framework makes distinction of "public" actors and "private"
actors. "Public" actors are those that have explicit registration in
the address book, while "private" actors can only be referenced by
other public actors which they have handles.

## Discussions

### Message-passing consistency

Consistency of message-passing to the same actor is ensured, but not
different ones. For example, in a process loop, if `A` emits messages
`X`, `Y`, `Z` to `B`, `C` and then `B` again. We ensure that `X` is
always processed before `Z`, but we make no promise of the execution
order of `Y`.

### Increased messages and invocation

This async design would require more message passing round compared
with sync contract model. The impact (both to execution and storage)
can be mostly limited by having a good schedule queue of process loop.
