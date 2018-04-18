
Handling of corner cases may be a bit different than in a programming language like `Java`.

Let's have a look at the following examples in order to understand the differences.
To make the analysis simpler, let's assume that there is only one `Motorbike` object stored in a Hazelcast Map.

| Id  | Query                                                  | Data state                            | Extraction Result   | Match |
| --- | ------------------------------------------------------ | ------------------------------------- | ------------------- | ----- |
|  1  | `Predicates.equal("wheels[7].name", "front-wheel")`    | `wheels.size() == 1`                  | `null`              | No    |
|  2  | `Predicates.equal("wheels[7].name", null)`             | `wheels.size() == 1`                  | `null`              | Yes   |
|  3  | `Predicates.equal("wheels[0].name", "front-wheel")`    | `wheels[0].name == null`              | `null`              | No    |
|  4  | `Predicates.equal("wheels[0].name", null)`             | `wheels[0].name == null`              | `null`              | Yes   |
|  5  | `Predicates.equal("wheels[0].name", "front-wheel")`    | `wheels[0] == null`                   | `null`              | No    |
|  6  | `Predicates.equal("wheels[0].name", null)`             | `wheels[0] == null`                   | `null`              | Yes   |
|  7  | `Predicates.equal("wheels[0].name", "front-wheel")`    | `wheels == null`                      | `null`              | No    |
|  8  | `Predicates.equal("wheels[0].name", null)`             | `wheels == null`                      | `null`              | Yes   |


As you can see, **no** `NullPointerException`s or `IndexOutOfBoundException`s are thrown in the extraction process, even
though parts of the expression are `null`.

Looking at examples 4, 6 and 8, we can also easily notice that it is impossible to distinguish which part of the
expression was null.
If we execute the following query `wheels[1].name = null`, it may be evaluated to true because:

* `wheels` collection/array is null.
* `index == 1` is out of bound.
* `name` attribute of the wheels[1] object is `null`.

In order to make the query unambiguous, extra conditions would have to be added, e.g.,
`wheels != null AND wheels[1].name = null`.

