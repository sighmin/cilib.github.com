---
layout: post
title: "Simplified domain string grammar"
description: "Updated grammar for domain sting parsing. The updated grammar results in a simpler parse for the domain string and clearer instance creation strategies."
category: updates
tags: [core, domain, grammar]
---

## The need to change domain string grammar
Defining a domain for use within a simulation is allowed within CIlib
as a simple string, such as:

    R(-10.0,3.0)^7,Z(0,1),B^2,R(0.0,5.0)

The notation is simple enough, but it is unfortunately difficult to work with
because the ',' token is overloaded to be both a separator for individual
dimensions within the domain definitions as well as being the separator for
lower and upper bound definitions within the dimension as well.

In order to parse the above, a fair amount of state needs to be maintained
to know the current "scope". This mutable state results is utter havoc with
concurrent usage of the domain parser. Two possible fixes could be applied to
resolve this issue:

1. Re-create the parser each time, ensuring that each thread has its own
   instance.
2. Make the parser stateless.

Option (1) seems reasonable but has a massive amount of effort for very little
gain, whereas option (2) means that the parser can not only be reused, but that
the concurrency issues should just disappear because no state is mangled.

As a result, we have reimplemented the parser to be as stateless as possible.
There is still some state within the parboiled parser library code that we
have made peace with.

## What has changed
The new parser now has two phases: (1) expansion and (2) transformation. The
expansion process is responsible to convert "exponent" strings into simpler
forms. As an example:

    R^7 => R,R,R,R,R,R,R

The transformation phase then converts the declared types into actual instances
that may be used within the algorithm. In order to differentiate the bounds
and dimension elements, the upper and lower bounds separator has been changed
from a ',' (comma) to a ':' (colon).

The new domain string is then:

    R(-10.0:3.0)^7,Z(0:1),B^2,R(0.0:5.0)

This is a minor change to the domain string representation, but the benefits
we have gained through this little modification are far too large to pass up.
To allow for the user to update the XML simulation specifications, clearer
error messages have been created with the simulation refusing to execute
without an updated domain string.
