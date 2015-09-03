## Table of Contents

- [General](#general)
- [Commas](#commas)
- [Comments](#comments)
- [Configurability](#configurability)
- [Consistency](#consistency)
- [`Context`](#context)
- [Conventions](#conventions)
- [Defaults](#defaults)
- [Dependency injection](#dependency-injection)
- [Documentation](#documentation)
- [DRY](#dry)
- [Equality](#equality)
- [Hilfingerisms](#hilfingerisms)
- [Hoisting](#hoisting)
- [Iteration](#iteration)
- [Line breaks](#line-breaks)
- [Loop invariants](#loop-invariants)
- [`for..in`](#forin)
- [Naming](#naming)
- [Optimizations](#optimizations)
- [Order](#order)
- [Semicolons](#semicolons)
- [References](#references)
- [Changelog](#changelog)

## General

1. Understand your language. Good C advice isn't necessarily good JavaScript advice. Avoiding recursion, for example, is bad advice for JavaScript programmers.
2. Understand your execution environment. If you're developing for the browser, explore the HTML5 APIs available to the runtime. Although not strictly part of the language itself (in the same way the UNIX system call interface isn't part of the C language), browser APIs are extremely useful if you want to do things like manipulating the history, storing lots of data on client side, making offline browsing of your app possible, and so on. If you're developing for the server, understand how processes and threads work, among other things. (See [Hilfingerisms](#hilfingerisms) for more on not being clueless.)
3. Read other people's code. Github is full of open source projects. Some of those projects have many thounsands of eyes on them and can teach valuable lessons on style and substance.

## Commas

1. Add commas at the ends of object property declarations. Do so even for the last one to aid diffing.

   ```js
   var obj = {
      a1: 1,
      a2: 2,
   };
   ```

## Comments

If the logic is necessarily complicated, don't hesitate to add comments to clarify the intent. Do not describe the implementation, for the code is the best documentation of that.

1. Two spaces after inline comments (`//`).
2. Block comments are used more as labels. Use `//` for comments where possible.
3. Don't add useless comments (e.g. `var i = 0;  // Let i be 0`).
4. Comments aren't a substitute for clear code.
5. Make sure comments are not out of date.
6. If you're familiar with Closure Compiler or JSDoc annotations, use them.
7. If you have a long piece of code (300 lines is super long), add a comment at the end of the closing brace to help the reader understand where it ends. For example, if you have a function like this:

   ```js
   var GraphCreator = function(svg, nodes, edges) {
       ...
   };
   ```
You can do something like the following:

   ```js
   var GraphCreator = function(svg, nodes, edges) {
       ...
   };  // end of GraphCreator
   ```

   ```js
   var GraphCreator = function(svg, nodes, edges) {
       ...
   };  // <-- GraphCreator
   ```

## Configurability

1. Don't embed long strings of text inside the main body of the code. Store them in *config* files and import them instead to localize changes to those files.
2. If there isn't a lot of configuration to warrant a separate file (less than 5 lines), move the configuration data to the top of the module.

## Consistency

1. Often there are neither prescriptions nor conventions to guide a choice of a name, placement of a function, depth of a namespace, etc. In those cases it's imperative that you choose something reasonable, document it, and use it consistently throughout your files.

## `Context`

`Context` is a global variable that acts as the root *namespace* for our objects and as a collection of *environment variables* for configuring our apps. About each role in turn.

**`Context` as Namespace.** The frontend core modules simply attach properties on `Context`, but as the modules grow, we're likely to confine those properties to their own namespaces.  In the near future `Context` should have this structure:

   ```js
   var Context = {

       Analytics: {
           assignments: [..],
           students: [..],
       },

       Core: {
           comments: [..],
           modules: [..],
       },

       Tooltips: {
           pageName: 'currentDraft',
           autoStart: false,
       },
   };
   ```

This way different modules will not clutter the global namespace with their own objects and avoid variable name *collisions*.

**`Context` as environment.** `Context` houses what is effectively a set of *environment variables* that we can set in Django's templates before we source a JS app. Consider the following template snippet:

   ```html
   <script>
       Context.user = '{{ user }}';
   </script>
   <script src="{% static 'js/bin/app.js' %}"></script>
   ```

There we assign the authenticated user's name to a variable on `Context` so that *app.js* may be able to do stuff with it. Logically, this is just like running commands in the shell:

   ```bash
   $ USER='john' node app.js
   ```

Username can then be accessed inside the running process by querying `process.env`:

   ```js
   console.log(process.env.user);  // => 'john'
   ```

Unfortunately we can't do something similar in the Browser:

   ```html
   <script src="{% static 'js/bin/app.js' %}" argv="user='{{user}}'"></script>
   ```
so we use `Context` to indirectly pass arguments to *app.js*.

## Conventions

1. Where the language lacks expressive power (e.g. constants, types), adhere to established conventions to make collaboration with your future self, your team, and the developer community at large easier.
2. Where the language provides alternative ways of composing programs, choose the more idiomatic versions over the less idiomatic ones. Even if you've spent most of your life squeezing maximum of efficiency out of the Linux kernel, it may be better to go with higher-order functions and closures instead of the good old for loops.

## Defaults

Defaults for variables of value type (e.g. numbers, strings, booleans) should usually be values of the corresponding type, whereas defaults for variables of reference type (viz. objects) should be `null`:

   ```js
   var name = '';
   var age = 0;
   var married = false;
   var friends = [];
   var cv = null;
   ```

Although `friends` points to an Array and is thus of a reference type, its default value should be the empty array so that we can simply test for `friends.length` to check emptiness. `cv` however, is better off being assigned `null` (this is a convention according to which `null` is assigned to all variables that expect an `Object` but somehow don't or can't have it).

## Dependency injection

1. If `obj1` depends on `obj2`, it's better to allow another object to create `obj2` and pass it (by value or by reference) to `obj1`. This may be automated (using, say, AngularJS).
2. Dependency injection allows dependencies to be swapped with minimal refactoring.
3. Dependency injection allows us to test modules by giving objects simpler versions of their dependencies that nevertheless expose the same interface as their full versions.

## Documentation

1. Before you start patching a framework or library you're using, be sure to read its documentation and source code. There may be a good reason you're not able to accomplish the thing you want to do.

## DRY

Whenever you find yourself typing the same or very similar piece of code, you're creating a refactoring problem: if any one of those pieces of code needs to be changed, the others have to be grepped and changed as well. If you change one and forget to change the others you can end up with outdated code that may no longer work. Code can lack DRYness in many ways and there are many measures you can take to keep your code DRY.

1. Name "magic numbers".

   Instead of embedding "magic numbers" in the logic:
   ```js
   setInterval(sync, 1000);  // Syncs stuff every second.
   ```

   abstract them out by naming them:
   ```js
   (1) var syncIntval = 1000;          // Name the literal
   (2) setInterval(sync, syncIntval);  // Use the name
   ```

   This move has many benefits (other than keeping the code DRY):

   1. The sync interval can be modified throughout the file by changing a single word.
   2. The assignment (2) can be placed in a *header* file and moved into *config/*.

2. Name recurring string literals

   Instead of embedding string literals in the logic:
   ```js
   $('#graph-select option:selected').val();
   ```

   abstract them out by naming them:
   ```js
   var $graphSel = $('#graph-select');       // Name the literal
   $graphSel.find('option:selected').val();  // Use the name
   ```

## Equality

1. Always use *strict equality* (`===` and `!==`) for comparing value types. Even if you know the types of the relata.

## Hilfingerisms

1. **"Read the fucking manual"**. The advice is very general. If you're using a real operating system, try `man` or `info` and understand the commands at your disposal. If you're using a good library or framework, it's likely to have great documentation online. Try Underscore's, Backbone's, and Django's documentations for examples of great-looking and useful docs.
2. **"Don't be clueless"**. Don't just copy and paste stuff without understanding from the internet (e.g. `rm -rf /`). Don't `curl` and shell out to scripts you didn't `cat`. Don't use IDEs that do things that you couldn't do without them. Understand your tools.
3. **"Be lazy"**. Whenever you find yourself repeating the same task, you might want to consider investing in some time to automate whatever it is that you're doing. Say you have to count the sum of the byes of all the files in your working directory. If you have only 10 files, you might be tempted to simply list the number of bytes for each file and add them in your head in under a minute. The correct, lazy thing, however, would be to spend several times that amount of time writing something more reusable using `stat`, `awk`, `sed`, or other tools.

## Hoisting

Variable declarations are *hoisted* in JavaScript, which means that no matter where you declare variables, the compiler is going to move them to the beginnings of the innermost functions they belong to. This:

   ```js
   for (var i = 0; i < MAX; i++) {...}
   ```

is equivalent to this:

   ```js
   var i;
   ...
   for (i = 0; i < MAX; i++) {...}
   ```

To avoid confusions due to the sharing of variables among different loops and their enclosing functions, declare all local variable at the top of those functions.

## Iteration

Although there are ordinary `for` and `while` loops in JavaScript:

   ```js
   // For
   var i;
   for (i = 0; i < MAX; i++) {...}

   // While
   var j = 0;
   while (j < MAX) {...; j++}
   ```

it is more idiomatic to use higher-order functions with closures. Here are some examples:

   ```js
   // Higher-order functions
   array.forEach(function(element) {...});
   array.map(function(element) {return ...});
   array.filter(function(element) {return ...});
   ```

One advantage of the latter is that instead of nesting loops inside each other, the higher-order functions can be *chained*. Consider the task of taking the sum of the squares of 1...5. Assume:

   ```js
   function add(m, n) {return m + n;}  // Add two numbers together
   function square(n) {return n * n;}  // Takes the square of a number
   ```

   ```js
   /* Imperative version (Bad) */
   var i;
   var sum = 0;
   var arr = [1, 2, 3, 4, 5];

   for (i = 0; i < arr.length; i++) {
       sum = add(sum, square(arr[i]));
   }
   ```

   ```js
   /* Functional version (Good) */
   var sum = [1, 2, 3, 4, 5].map(square).reduce(add);
   ```

## Line breaks

1. There should be an empty line at the end of every file to aid diffing.
2. Logically unrelated pieces of code should be separated by an empty line.

## Loop invariants

Loop invariants are conditions that hold before and are preserved by iterations of the loop. Advanced compilers are able to perform what's called *loop invariant code motion* (LICM), but it's a good idea to move loop invariant statements out of the bodies of loops for efficiency and understandability. The following, for example, is bad:

   ```js
   assignments.forEach(function(assignment) {
       var student = $studentSel.val();
       ...
   });
   ```

But this is good:

   ```js
   var student = $studentSel.val();
   assignments.forEach(function(assignment) {
       ...
   });
   ```

## `for..in`

The `for .. in --]` loop is problematic because it iterates over the entire *prototype chain* of the object (`--`) the keys (`..`) of which it's traversing. Consider the following code:

   ```js
   var father = {
       firstName: 'John',
       lastName: 'Doe',
       toString: function() {
           return this.firstName + ' ' + this.lastName;
       }
   };
   var son = {
       firstName: 'John Jr.'
   };

   father.toString();  // => "John Doe"
   son.toString();     // => "John Jr. Doe"
   ```

While `son` doesn't have its own `lastName` and `toString` properties, those are available on it because of JavaScript's scoping (or name-resolution) rules. The relevant rule here is:

> The value of `obj.atrr` is:
> 1. the first `attr` property on the *prototype chain* of `obj`, if one exists.
> 2. `undefined`, if no such property exists on the prototype chain.

If we wanted to loop over the keys on the `son`, we could try something like this:

   ```js
   for (key in son) {
     console.log(son[key]);  // => "John Jr.", "Doe", function father.toString()
   }
   ```
Not exactly what we wanted. To iterate over just the properties that are on the `son` itself, we could do:

   ```js
   for (var key in son) {
     if (son.hasOwnProperty(key) {
       console.log(son[key]);  // => "John Jr."
     }
   };
   ```

   or in a more compact way:
   ```js
   Object.keys(son).forEach(function(key) {
     console.log(son[key]);   // => "John Jr."
   });
   ```

## Naming

1. Names of jQuery nodes should start with a '$'.
   ```js
   var graph = $('#graph');   // Bad
   var $graph = $('#graph');  // Good
   ```

2. If you're using `doc` to refer to a document, don't use `doc` also to refer to documentation or doctors.

3. Use the name of an object to convey some idea of its type, if possible.

   ```js
   var userNames = ['John', 'Jane', ...];               // String[]
   var users = new Backbone.Collection({model: User});  // Backbone.Model[]
   ```
4. Don't use [reserved words](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Reserved_keywords_as_of_ECMAScript_6) as variables.

## Optimizations

1. "Premature optimization is the root of all evil" (Knuth)
2. Unless proven otherwise, assume that the compiler is more clever than you and don't do *microptimizations*. This means, don't do things like `++i` instead of `i++` because some people on StackOverflow think it performs better. Compilers have elaborate algorithms for optimizing such things, so don't get in the way (you might actually slow things down).
3. For accurate benchmarking use [benchmark.js](http://benchmarkjs.com) or some such statistical analysis tool.

## Order

1. If variable declarations have no natural order to them (e.g. variable `id` being dependent on variable `node` is an example of a natural order), order them alphanumerically.

   ```js
   // Bad
   var nodes = graph.getNodes();
   var edges = graph.getEdges();
   ```

   ```js
   // Good
   var edges = graph.getEdges();
   var nodes = graph.getNodes();
   ```

   The same convention holds for declarations of dependences in AMD modules. Here is an example.

   ```js
   // Bad
   define(['underscore', 'Backbone'], (_, Backbone) => ...);
   ```

   ```js
   define(['Backbone', 'underscore'], (Backbone, _) => ...);
   ```

   Although Backbone depends on underscore, all those dependencies are declared in the build file. Here their natural order doesn't need not be maintained.

## Semicolons

Certain statements (e.g. `let`, `import`/`export`, `break`) in JS [must](http://www.ecma-international.org/ecma-262/6.0/#sec-automatic-semicolon-insertion) be terminated with semicolons. That burden, however, is not entirely on the programmer. The engine follows an algorithm (called [Automatic Semicolon Insertion](http://www.ecma-international.org/ecma-262/6.0/#sec-rules-of-automatic-semicolon-insertion)) for inserting the missing semicolons. But the algorithm is [greedy](https://en.wikipedia.org/wiki/Greedy_algorithm), so it will insert a semicolon at the end of the line only if it cannot consume any more meaningful tokens. This may result in surprising behavior. To avoid being surprised, if you're not sure whether a semicolon will be inserted by ASI, don't skip them.

1. Don't add semicolons after *function declarations*.

    ```js
    function foo() {...};  // Bad
    function foo() {...}   // Good
    ```

2. Add semicolons when naming *function expressions*.

    ```js
    var foo = function() {...}   // Bad
    var foo = function() {...};  // Good
    ```

## References

- The [ECMAScript 2015 Specification](http://www.ecma-international.org/publications/standards/Ecma-262.htm)
- Analytics Table for Courses (Pull Request #138)

