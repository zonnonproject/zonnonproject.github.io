---
layout: page
title: Zonnon Language
---

[Zonnon](https://en.wikipedia.org/wiki/Zonnon) is a general-purpose programming language in the Pascal, Modula-2 and Oberon family.
Its conceptual model is based on modules, objects, definitions and implementations.
Zonnon offers a computing model based on active objects with their interaction defined by syntax controlled dialogs.  

{% highlight oberon %}
module HelloWorld;

begin
  writeln('Hello, GitHub!')
end HelloWorld.
{% endhighlight %}

Please refer to [Zonnon Language Report] for the exact language specification.

Here we will do a brief introduction by example.

# Program Structure

{% highlight oberon %}
module Example21;
var
  x, y, sum: integer;
begin
  write("Input X: "); readln(x);
  write("Input Y: "); readln(y);
  sum := x + y;
  writeln("X + Y = ", sum);
end Example21.
{% endhighlight %}

# Data Types

{% highlight oberon %}
module Example31;
type
  word = integer{16};
var
  i: integer;
  j: word;
begin
  i := max(integer); (* Max value for integer type *)
  writeln(i, " ", min(integer));
  j := max(word);    (* Max value for integer{16} type *)
  writeln(j, " ", min(word));
end Example31.
{% endhighlight %}

Output

{% highlight ksh %}
2147483647     -2147483648
32767          -32768
{% endhighlight %}

{% highlight oberon %}
module Example32;
var b: boolean;
begin
  b := 2 * 2 = 4;
  writeln(" 2 * 2 = 4 is ", b);
end Example32.
{% endhighlight %}

Output

{% highlight ksh %}
2 * 2 = 4 is true
{% endhighlight %}

{% highlight oberon %}
module Example33;
var
  s: set;
  s33: set { 16 };
  s128: set { 64 };
begin
  s := { 1 };
  s := { min(set) .. max(set) };
end Example33;
{% endhighlight %}

{% highlight oberon %}
module Example34;
var ch:char;
begin
  ch := 100X;
  writeln(ch);
end Example34.
{% endhighlight %}

{% highlight oberon %}
module Example35;
type
  NumberKind = (Bin, Oct, Dec, Hex);
var
  b: NumberKind;
begin
  b := NumberKind.Bin;
end Example35;
{% endhighlight %}

# Expressions

{% highlight oberon %}
module Example41;
type
  Figure: (Star, Rectangle, Circle, Ellipse);

var
  a, b: real;
  x, y: integer;
  myfigure: Figure;
  align: ( Top, Bottom, Left, Right );
begin
  align := align.Right;
  writeln( integer( align ) );
end Example41.
{% endhighlight %}

{% highlight oberon %}
const
  N = 10;
  LIMIT = 2*N – 1;
  FULLSET = {min(set)..max(set)};
  TEXT1 = "Double quoted string";
  TEXT2 = ‘Single quoted string‘;
  SEPARATOR = ‘*‘;
{% endhighlight %}

{% highlight oberon %}
module Example331;
var
  s: set{64};
  n: integer;
begin
  s := {}; (* Empty set *)
  writeln("Input number < 64. 0 - to quit.");
  repeat
    write("> "); readln(n);
    incl(s,n);
  until n = 0;
  writeln;
  for n:= 1 to 63 do
    if n in s then
      write(n:3);
    end
  end;
  writeln;
end Example331
{% endhighlight %}

{% highlight oberon %}
module Example444;
var
  s, t:string;
  ch:char;
begin
  t := s + ".txt";
  s := t[2..5];
  s := t[0..#t];
  ch := s[1];
end Example444.
{% endhighlight %}

# Standard Library

{% highlight oberon %}
module Example432b;
import System.Math as Math;
begin
  writeln(Math.Abs(-10));
end Example432b.
{% endhighlight %}

{% highlight oberon %}
module Example102;
import System, System.IO;
type
  SW = System.IO.StreamWriter;
var
  sw: SW;
  fn: System.String;
begin
  fn := "myfile.txt";
  sw := new System.IO.StreamWriter(fn, false);
  sw.Write("Text to be written to file");
  sw.Close;
  readln;
end Example102.
{% endhighlight %}

# Statements

{% highlight oberon %}
module example522;
var
  a, b, m: integer;
begin
  write("A: "); readln(a);
  write("B: "); readln(b);
  if a > b then
    m := a
  else
    m := b
  end;
  writeln("Max of a and b = ", m);
end example522.
{% endhighlight %}

{% highlight oberon %}
module example523;
var
  a: char;
begin
  write("Input char: "); readln(a);
  case a of
  "0".."9":
    writeln('Digit')
  else
    writeln('Not a digit')
  end
end example523.
{% endhighlight %}

{% highlight oberon %}
module example524;
var
  num, dig: integer;
begin
  write("Input integer: ");
  readln(num);
  dig := 0;
  while num # 0 do
    inc(dig);
    num := num div 10
  end;
  writeln("Number of digits: ", dig);
end example524.
{% endhighlight %}

{% highlight oberon %}
module example525;
var length: integer;
begin
  repeat
    write("Input length( > 0): "); readln(length);
  until length > 0;
  writeln("Accepted length: ", length);
end example525.
{% endhighlight %}

{% highlight oberon %}
module example526;
var
  m, n, a, i: integer;
begin
  write("Input a 4-digit number: ");
  readln(n);
  m := n;
  a := 0;
  for i := 1 to 4 do
    a := a * 10 + m mod 10;
    m := m div 10;
  end;
  if a = n then
    writeln("Is a palindrome")
  else  
    writeln("Not a palindrome")
  end;
end example526.
{% endhighlight %}

{% highlight oberon %}
module example527;
import System;
var ch: char; h, r: real;
begin
  loop
    write("Input hight h:"); readln(h);
    write("Input r:");
    readln(r);
    writeln("Cone surface:", System.Math.PI * h * r * r / 3);     
    write("Done? (Y/N):"); readln(ch);   
    if (ch = 'Y') or (ch = 'y') then exit end;
    writeln;
  end;
end example527.
{% endhighlight %}

{% highlight oberon %}
module example610;
const N = 100;
var
  i: integer;
  a: array N of integer;
begin
  for i := 0 to N - 1 do a[i] := i end;
  for i := 0 to N - 1 do write(a[i]:4) end;
end example610.
{% endhighlight %}

{% highlight oberon %}
module example610a;
type
  Vector = array * of integer;
var
  i, n: integer;
  a: Vector;
begin
  write("Number of elements: "); readln(n);
  a := new Vector(n);
  for i := 0 to len(a) - 1 do
    write("a[",i:2,"]: "); read(a[i])
  end;
  writeln;
  for i := 0 to len(a) - 1 do
    write(a[i]:3);
  end;
  writeln;
end example610a.
{% endhighlight %}

# Data Structures

{% highlight oberon %}
module Example73;

record {ref} RefNumber;
  val: integer;
end RefNumber;

record Number;
  val: integer;
end Number;

var
  a, b: RefNumber;
  c, d: Number;

begin
  a := new RefNumber;
  a.val := 10;
  b := a; (* Reference assignment *)
  b.val := 11;
  writeln(a.val, " = ", b.val); (* 11 = 11 *)

  c.val := 15;
  d := c; (* Value assignment  *)
  d.val := 16;
  writeln(c.val, " # ", d.val); (* 15 # 16 *)
end Example73.
{% endhighlight %}

# Subprograms

{% highlight oberon %}
module Example91;
  procedure Maximum(a, b: integer; var m: integer);
  begin
    if a > b then
      m := a
    else
      m := b
    end
  end Maximum;
var
  a, b, m: integer;
begin
  write("A = "); readln(a);
  write("B = "); readln(b);
  Maximum(a, b, m);
  writeln("max( A, B ) = ", m);
end Example91.
{% endhighlight %}

{% highlight oberon %}
module Example92;
  procedure Maximum(a, b: integer):integer;
  begin
    if a > b then
      return a
    else
      return b
    end
  end Maximum;
var
  a, b: integer;
begin
  write("A = "); readln(a);
  write("B = "); readln(b);
  writeln("max( A, B ) = ", Maximum(a,b));
end Example92.

{% endhighlight %}

# Program composition

Read [Program Composition in the Small](https://blog.mitin.ch/program-composition-in-small.html)

# Concurrency
Read [A note on activities, protocols and communication](https://blog.mitin.ch/activities-protocols-communication.html)

[Zonnon Language Report]: https://zonnon.org/files/Zonnon-Language-Report_v03r01_35_y051214.pdf
