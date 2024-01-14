---
title: Program composition in the small
---
The purpose of this post is to clarify one aspect of compositional model in Zonnon and in particular the use of import directive in the context of object construction. There are four kinds of building blocks: definition, implementation, object and module. And there are four types of relations between them: import, aggregation, implementation and refinement. Both import and aggregation are represented with keyword import.

We will use a simple example to illustrate how objects are composed.

A definition defines an abstract interface possibly comprising variable declarations, method signatures, and syntactically defined protocols. For example the following declaration defines an interface containing one procedure.

{% highlight oberon %}
definition {public} A.D;

  procedure P;

end D.
{% endhighlight %}

An object may implement any number of definitions.

Before continuing, I want to clarify the use of namespaces. Definition D is defined within namespace A. Identifier D should be always used with qualifying namespace unless renamed by an import clause. In other words if there is another unit defined also within namespace A uses D it needs to qualify D with A anyway.

In the example below object A.B.O implements definition A.D and procedure P within object A.O.O implements procedure A.D.P.

{% highlight oberon %}
object {ref} A.B.O implements A.D;

  var {public}
    a: object{A.D};

  procedure {public} P implements A.D.P;
  begin
    writeln("A.D.P")
  end P;

end O.
{% endhighlight %}

Implements clause implies an implicit import of A.D. in the example above. However we still can write an explicit import. Explicit import allows to give a local nickname for the imported unit. Example below is an equivalent to the example above.

{% highlight oberon %}
object {ref} A.B.O implements D;

  import A.D as D;

  var {public}
    a: object{D};

  procedure {public} P implements D.P;
  begin
    writeln("A.D.P")
  end P;

end O.
{% endhighlight %}

Module units can import declarations from other modules. The import clause shows explicit dependencies between modules. This also applies to the standalone definition, implementation and object units.

As you can see from the example below the transitivity rule does not apply here. Both the object type and the definition must be imported if they are used in the module.

{% highlight oberon %}
module Main;

import A.B.O, A.D;

var
  o: object{A.D};

begin
  o := new A.B.O;
  o.P
end Main.
{% endhighlight %}

To clarify the case with ‘topmost’ level definitions, objects and implementation note that a standalone object declaration is equivalent to the type being declared in an anonymous module.

{% highlight oberon %}
object T;

  . . . .

end T.
{% endhighlight %}

is shorthand for

{% highlight oberon %}
module ;

  type T = object . . . end T;

end .
{% endhighlight %}

A definition can refine another definition by adding services. Originally refinement also meant ability to omit and modify services, but this no not supported.

In the example below a definition K is a refinement of definition D. It adds a new procedure U.

{% highlight oberon %}
definition {public} B.K refines A.D;

  procedure U;

end K.
{% endhighlight %}

Any object that implements definition B.K must implement both procedures P and U.

{% highlight oberon %}
object {ref} A.B.O2 implements B.K;

  procedure {public} P implements B.K.P;
  begin
    writeln("B.K.P")
  end P;

  procedure {public} U implements B.K.U;
  begin
    writeln("B.K.U")
  end U;

end O2.
{% endhighlight %}

Note that procedure P implements B.K.P; and not A.D.P as P is visible in the scope of definition K and definition D is not visible in O2 at all.

An implementation defines an aggregation of variable and method implementations intended for reuse. In the example below there is a pair of definition and implementation. Implementation

{% highlight oberon %}
definition {public} A.T;

  procedure S;

  procedure L;

end T.

implementation A.T;

  procedure S implements A.T.S;
  begin
    writeln("A.T.S")
  end S;

end T.
{% endhighlight %}

Implementation A.T provides a default implementation for method S which can be reused. The following example illustrates an implicit aggregation. Object O3 implements definitions B.K and A.T and implicitly aggregates implementation A.T which contains procedure S.

{% highlight oberon %}
object {ref} A.B.O3 implements B.K, A.T;

  var {public}

  procedure {public} P implements B.K.P;
  begin
    writeln("B.K.P")
  end P;

  procedure {public} U implements B.K.U;
  begin
    writeln("B.K.U")
  end U;

  procedure {public} L implements A.T.L;
  begin
    writeln("A.T.L")
  end L;

end O3.
{% endhighlight %}

An interface is a type for a postulated object composed from one or more definitions. In the example below the construct
{% highlight oberon %}
t: object{A.T};
{% endhighlight %}
declares a variable of a generic object type with a specifier that it implements the definition A.T.
The other example
{% highlight oberon %}
object{A.T, B.K};
{% endhighlight %}
defines an interface type that implements both definitions A.T and B.K.

{% highlight oberon %}
module Main;

import A.B.O3, B.K, A.T;

var
  o: A.B.O3;
  kt: object{A.T, B.K};
  t: object{A.T};

begin
  o := new A.B.O3;
  t := o;
  t.S;
  t.L
end Main.
{% endhighlight %}
