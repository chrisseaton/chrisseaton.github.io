---
layout: article
title: Ruby Objects as C Structs and Vice Versa
author: Chris Seaton
date: 6 March 2018
copyright: Copyright © 2018 Chris Seaton.
---

In TruffleRuby we run Ruby C extensions using an interpreter with a JIT that is
built very much like our interpreter and JIT for Ruby.

We do this for three key reasons - it allows us to pass Ruby objects into C
without converting them to a native representation, it allows us to add inline
caches for Ruby C API calls, and it allows us to intercept C operations and
redirect them to work with our internal representations of Ruby objects.

We've done this in order to implement the Ruby C API with better performance
than existing efforts like JRuby and Rubinius, but we've also found that it's
allowed us to do some fun new things that we didn't expect.

This blog post shows one of those fun new things - using Ruby objects as if they
were a C `struct`, just by casting, and the other way around - using a C
`struct` as if it was a Ruby object.

Consider a C extension function that calculates the magnitude of a Ruby vector
object. It's cumbersome to access a Ruby object from C - you need to use
functions to call the accessor methods, and you need to convert the types from
Ruby to C. This is the only way to access the object in other implementations of
Ruby.

```ruby
class Vector
  
  attr_reader :x, :y
  
  def initialize(x, y)
    @x = x
    @y = y
  end
  
end
```

```c
VALUE magnitude(VALUE self, VALUE vector) {
  double x = NUM2DBL(rb_funcall(vector, rb_intern("x"), 0));
  double y = NUM2DBL(rb_funcall(vector, rb_intern("y"), 0));
  return DBL2NUM(sqrt(x*x + y*y));
}
```

In TruffleRuby we can instead define a C `struct` to mirror the Ruby object, and
then cast the Ruby object to a pointer to this `struct` and read the fields as
normal. These field reads will be intercepted by our C interpreter and turned
back into the correct Ruby calls to the accessor methods. The types are
converted for you as well.

```c
struct Vector {
  double x;
  double y;
};

VALUE magnitude(VALUE self, VALUE vector) {
  struct Vector *v = (struct Vector *)vector;
  double x = v->x;
  double y = v->y;
  return DBL2NUM(sqrt(x*x + y*y));
}
```

You can also do this the other way around. Consider a C extension function that
returns the date. It creates an `OpenStruct` and sets `year`, `month` and `day`
fields. Again this is pretty cumbersome - you need to manually convert the types
and you need to call Ruby methods using the C API.


```c
VALUE date(VALUE self) {
  time_t seconds = time(NULL);
  struct tm *date = localtime(&seconds);
  VALUE day = rb_funcall(
      rb_const_get(rb_cObject, rb_intern("OpenStruct")),
      rb_intern("new"), 0);
  rb_funcall(day,
      rb_intern("year="), 1,
      INT2FIX(1900 + date->tm_year));
  rb_funcall(day,
      rb_intern("month="), 1,
      INT2FIX(1 + date->tm_mon));
  rb_funcall(day,
      rb_intern("day="), 1,
      INT2FIX(date->tm_mday));
  return day;
}
```

In TruffleRuby you can define a C `struct`, allocate an instance of it, and then
just pass it back into Ruby with no extra work where it will appear as a Ruby
object.

```c
struct Day {
  int year;
  int month;
  int day;
};

VALUE date(VALUE self) {
  time_t seconds = time(NULL);
  struct tm *date = localtime(&seconds);
  struct Day *day = (struct Day *)malloc_special();
  day->year = 1900 + date->tm_year;
  day->month = 1 + date->tm_mon;
  day->day = date->tm_mday;
  return day;
}
```

```ruby
day = date
puts "#{day.year}-#{day.month}-#{day.day}"
```

I've defined that `malloc_special` function for this demo - it actually just
creates an `OpenStruct` as before, and then the structure field writes in it are
intercepted and turned into Ruby method calls.

```c
void *malloc_special() {
  return rb_funcall(
    rb_const_get(rb_cObject, rb_intern("OpenStruct")),
    rb_intern("new"), 0);
}
```

This technique of intercepting C `struct` reads and writes is how we implement
parts of the Ruby C API like `RData` (the general purpose user data container).
When you cast a Ruby object in TruffleRuby to `RData *` and read the `data`
field it actually redirects that to read an instance variable that holds your
user data.

We think is better than other alternative technique that JRuby had to use which
is to allocate a separate native structure which then has to use a handle to
refer back to the managed object. And it also enables the cool demo shown in
this blog post.

```c
struct RData {
  struct RBasic basic;
  void (*dmark)(void*);
  void (*dfree)(void*);
  void *data;
};
```

We only plan to use this technique internally for the cases like `RData` - we'd
recommend people normally keep writing C extensions in a way that's compatible
with MRI.

This is all possible because in our C interpreter instead of the `struct` field
reads just being a load from an address in memory we can instead insert any
logic we want. There's even an inline cache here so the call to the Ruby method
is fast and can be inlined.

You can try this demo yourself - see the
[GitHub repository](https://github.com/chrisseaton/struct-blog-post).

{% include jrubytrufflelinks.html %}
