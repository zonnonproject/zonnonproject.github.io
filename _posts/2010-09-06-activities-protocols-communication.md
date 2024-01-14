---
title: A note on activities, protocols and communication
---
The purpose of this post is to explain by example the concurrency model in Zonnon. The description of this model one can find in the Zonnon Language report.

The primary building block for concurrency is activities. Activities are used both for adding behavior to objects and for implementing interobject communication. Syntactically activities resemble procedures and are encapsulated in their host object. However, the difference lies in the runtime model. When a procedure is called, possibly with some parameters, it blocks the caller and runs on account of the caller’s thread. Upon completion it returns both return parameters and control to the caller. When an activity is called, possibly with some initial parameters according to its signature, a separate execution thread is created, and the caller is not blocked. Each activity implements a communication protocol (a kind of “contract”) that defines an exchange of tokens (a “dialog”) in terms of a formally defined syntax in EBNF.

In the following example, the body of module Main creates an instance a of a local activity of type A within its scope and interacts according to protocol P.

{% highlight oberon %}
object {actor} Main;

protocol P = ( N,F,
  prot = ({N ?integer} | F)
);

activity A implements P;
var
  i: integer;
  cmd: P;
begin
  i := 0;
  accept cmd;
  while cmd # P.F do
    return i;
    inc(i);
    accept cmd
  end
end A;

var
  i, next: integer;
  a: activity{P};
begin
  a := new A;
  for i:=1 to 10 do
    next := a(P.N);
    writeln(next)
  end;
  a(P.F)
end Main.
{% endhighlight %}

The caller uses a method call notation to send data (“messages”) to the callee and assignment logic to receive data. Statements accept and return are used on the callee side. In principle, the caller (client) is anonymous for the callee.

Protocols for activities correspond to signatures for procedures. A procedure can be regarded as an activity that accepts parameters only once in the beginning and returns results only once at the end. Both synchronous and asynchronous procedure calls can be modelled, where in the second case a “future” variable is used instead of the assignment logic.

Within a parent scope a full hierarchy of child activities may be created. If the parent scope is marked with a fence modifier, then its end by definition takes the role of an execution barrier that must not be crossed before all child activities have terminated. The scope of the calling routine is an implicit fence.

Objects and modules with at least one activity must be marked with modifier ‘actor’ or ‘protected’. If object is marked with ‘actor’ then it cannot have public procedures. In other words the (interoperability) interface of an actor replaces the method table interfaces of ordinary objects. It is the set of protocol-based activities that it exposes. Protected objects may mix activities and public methods.

The following example shows two communicating actors. Whenever a dialog is created by some caller actor, an activity (a kind of an agent) in the partner actor is launched. Actor Car features an intrinsic activity that implements driving into ParkingLot. This activity instantiates an agent activity within the scope of ParkingLot. An agent activity is defined within the scope of ParkingLot and therefore has access to the fields of ParkingLot. The two activities run in separate threads and communicate via message passing in the clothes of a dialog and using the tokens of protocol Service as messages.

{% highlight oberon %}
protocol Service = (
  Request, Allow, Reject, Enter, Leave, Wait,
  park = [Enter] Leave,
  waitorleave = Wait ?Allow park | Leave,
  dialog = Request (?Allow park | ?Reject waitorleave)
).

module {actor} ParkingLot;
import Service;

var
  capacity: integer;

activity {public} Serve implements Service;
var
  cmd: Service;
  enter: boolean;
begin
  accept cmd; (*Request*)
  enter := true;
  if capacity=0 then
    return Service.Reject;
    accept cmd; (*Wait or no wait*)
    if cmd = Service.Wait then
      await capacity > 0
    else
      enter := false
    end
  end;
  if enter then
    dec(capacity); (*Book*)
    return Service.Allow;
    accept cmd; (*Enter or Leave*)
    if cmd = Service.Enter then
      (*car stays in the lot*)
      accept cmd (*Leave*)
    end;
    inc(capacity)
  end
end Serve;

begin
  capacity := 2;
end ParkingLot.

object {ref, protected} Car;
import ParkingLot as PL, Service;

activity {public} Drive;
var
  parking: PL.Serve;
  ans: Service;
begin
  writeln("start");
  parking := new PL.Serve;
  ans := parking(Service.Request);
  if ans # Service.Allow then
    (*Ask to wait forever*)
    writeln("try again");
    ans := parking(Service.Wait)
  end;
  parking(Service.Enter);
  writeln("parked");
  (*stay in the parking lot*)
  await 100; (*sleep*)
  parking(Service.Leave);
  writeln("left")
end Drive;

begin
  new Drive;
end Car.

module Main;
import Car;
var i: integer;
begin
  for i:= 1 to 5 do new Car end
end Main.
{% endhighlight %}

Note that caller activities refer to callees via their instance name, while callers are anonymous from the callee's point of view. In our example, a reference ans to activity Serve is used by the caller to identify the communication.

The caller and the callee exchange tokens according to the protocol/ syntax specified by the callee. It is said that activity Serve implements a “parser” for protocol Service.

In the example above protocol Service defines an enumeration of tokens Request, Allow, Reject, Enter, Wait and Leave which are used as tokens or messages in the communication. In turn, the communication protocol is defined via three EBNF productions: park, waitorleave and dialog. Note that the question mark used in front of a token changes the direction of the message. Production park defines optional sending of token Enter and then mandatory sending of token Leave from caller to callee. Production dialog obliges the caller to send a Request token and then the callee to reply either with Allow or with Reject. If the callee replies with Allow then the continuation of the dialog is defined with production park. If the callee replies with Reject then the continuation of the dialog is defined with production waitorleave.

Note that in protocol definitions

* Protocol syntaxes must be deterministic and context-free.
* Recursion of productions is not allowed.
* In repetitions there must be at least one message in both directions. Since send operations are non blocking, this is needed to calculate buffer sizes statically and also to prevent buffer overflows. As an example, messages like {string} are not allowed, where {string ?OK} and {string string ?OK} are.
* Alternatives can start with transmissions in one direction only. For example, (A &#x7c; ?B) is not allowed. At each point in the protocol it must be clear whether the callee or the caller makes the decision. Our example can be rewritten as (A &#x7c; C ?B) if the decision is to be made by the caller or as (?C A &#x7c; ?B) if the callee decides. By the same reason, (string [string] ?OK) is not allowed, whereas (string [string] END ?OK) is perfectly fine.

Compliance of the transmitted data with the protocol is checked at runtime with a finite state automata and an exception can be raised in case of any deviation. As we emphasise again that validating of a communication protocol with respect to its formal declaration in a way corresponds to signature type checking in the case of procedure calls compiler tries to enforce protocol statically. Unfortunately a full protocol validation is not feasible in the general case, and it would require sophisticated model checking concepts. However without this one would not be able to write code productively.

This static validation procedure is the main reason why references of protocol types are limited to local variables and it is impossible to create a copy of this reference. There is one exception to that. It is allowed to pass a protocol reference in a procedure call. This does not affect the analysis as it can be replaced with inlining of the called procedure. Recursion of this calls in not allowed.

It is also allowed to pass a protocol reference to another activity within some dialog. This operation is allowed only for references to activities that have been assigned but not yet launched. It is necessary to ensure that the state of the protocol is known. The transfer operation works as destructive reading that is, the variable becomes unassigned and its value becomes invalid.

Local variables that are used for protocols in the language can be of some protocol type or of a concrete activity type. References of some protocol type can refer to any activity that implements this protocol type.

Instead of a conclusion a few words about the consistency model. Activities express potential parallelism. However it does not always makes sense to actually run them in parallel. Our aim is to provide two options: expressing desirable concurrency (on a very fine grain level) and to express unconstrained concurrency on coarse grain level. Activities that belong to different active objects can always run in parallel. Activities declared within one active object share the object state and may conflict.

We use a simple consistency rule to specify the “visible” behaviour based on the notion of atomic sequences. We define atomic sequence as a trace in the control flow of the program (possibly including method calls) that does not contain any blocking instructions (e.g. receiving a message, waiting for a condition or sleeping for a certain period of time).

Then we use a simple sequential consistency rule saying that for any execution order of a group of atomic sequences within one active object there is a sequential exclusive execution order that leads to the same result.

Atomic sequences may have been assigned a precondition that schedules the execution.

In ParkingLot example discussed before we can find, for example, following atomic sequences.

In the following example after receiving a message and until the next blocking statement the code is executed exclusively:

{% highlight oberon %}
  accept cmd; (*Blocking receive*)
  enter := true;
  if capacity = 0 then
    return Service.Reject;
    (*The next instruction is blocking*)
{% endhighlight %}

Since there is a branch the other trace when condition is false is also an atomic sequence:

{% highlight oberon %}
  accept cmd; (*Blocking receive*)
  enter := true;
  if capacity # 0 then
    return Service.Reject;
  if enter then
    dec(capacity); (*Book*)
    return Service.Allow;
    (*The next instruction is blocking*)
{% endhighlight %}

For example this trace is also atomic:

{% highlight oberon %}
  await capacity > 0; (*Blocking until condition is true*)
  if enter then
    dec(capacity); (*Book*)
    return Service.Allow;
    (*The next instruction is blocking*)
{% endhighlight %}

which ensures that capacity will be decremented only when it is greater than 0. If condition becomes eventually true then the activity eventually will be rescheduled. These were examples of conditional atomic sequence.

A body of objects and modules is always scheduled first, so it is safe to use the beginning of the body (before the first blocking statement) for initialization as a constructor. This is an unconditional atomic sequence:

{% highlight oberon %}
begin
  capacity := 2;
end ParkingLot.
{% endhighlight %}


The following atomic sequence will be scheduled not earlier than 100 milliseconds after the call of await:

{% highlight oberon %}
await 100; (*sleep*)
parking(Service.Leave)
{% endhighlight %}