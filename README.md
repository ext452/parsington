[![](https://img.shields.io/maven-central/v/org.scijava/parsington.svg)](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.scijava%22%20AND%20a%3A%22parsington%22)
[![](https://travis-ci.org/scijava/parsington.svg?branch=master)](https://travis-ci.org/scijava/parsington)

# Parsington

Parsington is an infix-to-postfix (or infix-to-syntax-tree) expression parser
for mathematical expressions written in Java. It is simple yet fancy, handling
(customizable) operators, functions, variables and constants in a similar way
to what the Java language itself supports.

Parsington is part of the [SciJava](http://scijava.org/) project
for scientific computing in Java.

## Rationale

Expression parsers are as old as the hills; what makes this one different?

* __No dependencies.__
* __[Available on Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.scijava%22%20AND%20a%3A%22parsington%22).__
* __Permissive BSD-2 license.__ See [LICENSE.txt](LICENSE.txt).
* __Separation of concerns.__ Parsington is a _parser_, not an _evaluator_.
  Once you have the postfix queue and/or syntax tree, what you do with it is
  your business (though there is a [small evaluation API in the eval
  subpackage](src/main/java/org/scijava/parse/eval) if that appeals to you).
  In general, there is no assumption that your variables will consist of any
  particular data type, numerical or otherwise.
* __Clean, well-commented codebase with unit tests.__ 'Nuff said!

## History

The [ImageJ Ops](https://github.com/imagej/imagej-ops) project needed an
expression parser so that it could be more awesome. But not one limited to
primitive doubles, or one that conflated parsing with evaluation, or one
licensed in a restrictive way. Just a simple infix parser: a nice shunting yard
implementation, or maybe some lovely recursive descent. Something on GitHub,
with no dependencies, available on Maven Central. But surprisingly, there was
only tumbleweed. So Parsington was born, and all our problems are now over!

## Usage

In your POM `<dependencies>`:
```xml
<dependency>
  <groupId>org.scijava</groupId>
  <artifactId>parsington</artifactId>
  <version>1.0.3</version>
</dependency>
```

### Postfix queues

To parse an infix expression to a postfix queue:
```java
LinkedList<Object> queue = new ExpressionParser().parsePostfix("a+b*c^f(1,2)'");
// queue = [a, b, c, f, 1, 2, (2), <Fn>, ^, ', *, +]
```

### Syntax trees

To parse an infix expression to a syntax tree:
```java
SyntaxTree tree = new ExpressionParser().parseTree("a+b*c^f(1,2)'");
```
```
       +-------+
       |   +   |
       +---+---+
           |
    +------+------+
    |             |
+---+---+     +---+---+
|   a   |     |   *   |
+-------+     +---+---+
                  |
           +------+------+
           |             |
       +---+---+     +---+---+
       |   b   |     |   '   |
       +-------+     +---+---+
                         |
                         |
                     +---+---+
                     |   ^   |
                     +---+---+
                         |
                  +------+------+
                  |             |
              +---+---+     +---+---+
              |   c   |     | <Fn>  |
              +-------+     +---+---+
                                |
                         +------+------+
                         |             |
                     +---+---+     +---+---+
                     |   f   |     |  (2)  |
                     +-------+     +---+---+
                                       |
                                +------+------+
                                |             |
                            +---+---+     +---+---+
                            |   1   |     |   2   |
                            +-------+     +-------+
```

### Evaluation

To evaluate an expression involving basic types:
```java
Object result = new DefaultEvaluator().evaluate("6.5*7.8^2.3");
```

### Interactive console

There is also an [interactive console
shell](src/main/java/org/scijava/parse/Main.java) you can play with.

Run it easily using [jrun](https://github.com/ctrueden/jrun):
```
jrun org.scijava:parsington
```

Or run from source, after cloning this repository:
```shell
mvn
java -jar target/parsington-*-SNAPSHOT.jar
```

Simple example invocations with the console's default evaluator:
```
> 6.5*7.8^2.3
732.3706691398969 : java.lang.Double
> 2*3,4
      ^
Misplaced separator or mismatched groups at index 4
```

The `postfix` built-in function lets you introspect a parsed postfix queue:
```
> postfix('6.5*7.8^2.3')
6.5 : java.lang.Double
7.8 : java.lang.Double
2.3 : java.lang.Double
^ : org.scijava.parse.Operator
* : org.scijava.parse.Operator
> postfix('[1, 2f, 3d, 4., 5L, 123456789987654321, 9987654321234567899]')
1 : java.lang.Integer
2.0 : java.lang.Float
3.0 : java.lang.Double
4.0 : java.lang.Double
5 : java.lang.Long
123456789987654321 : java.lang.Long
9987654321234567899 : java.math.BigInteger
[7] : org.scijava.parse.Group
> postfix('f(x, y) = x*y')
f : org.scijava.parse.Variable
x : org.scijava.parse.Variable
y : org.scijava.parse.Variable
(2) : org.scijava.parse.Group
<Fn> : org.scijava.parse.Function
x : org.scijava.parse.Variable
y : org.scijava.parse.Variable
* : org.scijava.parse.Operator
= : org.scijava.parse.Operator
> postfix('math.pow(q) = q^q')
math : org.scijava.parse.Variable
pow : org.scijava.parse.Variable
. : org.scijava.parse.Operator
q : org.scijava.parse.Variable
(1) : org.scijava.parse.Group
<Fn> : org.scijava.parse.Function
q : org.scijava.parse.Variable
q : org.scijava.parse.Variable
^ : org.scijava.parse.Operator
= : org.scijava.parse.Operator
```

The `tree` function is another way to introspect, in syntax tree form:
```
> tree('math.pow(q) = q^q')
 '='
 - '<Fn>'
  -- '.'
   --- 'math'
   --- 'pow'
  -- '(1)'
   --- 'q'
 - '^'
  -- 'q'
  -- 'q'
 : org.scijava.parse.SyntaxTree
```
