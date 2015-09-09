## Review of [PR 151] ("topic modeler")

In the [Stylistic issues](#stylistic-issues) section I talk about linting, in [Substantial issues](#substantial-issues) you'll find a categorized list of concerns/suggestions (often general, big picture observations). In [Suggestions](#suggestions) you'll find concrete suggestions for improving the code. Admittedly, some of the suggestions are non-normative (using `forEach` instead of C-style `for` loops), but I strongly believe that functional idioms are more readable, less error prone, and no less efficient than their imperative equivalents.

In [Questions](#questions) I've listed some questions (I think currently it's a single item list), in [Todo](#todo) I've listed some more things to do (it's kinda meta, unlike [Suggestions](#suggestions)). In [Next steps](#next-steps), I mention some things to be done in the near future. And [References](#references) contains links to resources used to prepare this document (currently just the JS Substance Guide).

*If you find typos or have concerns/corrections, email/pull request.*

### Table of Contents

- [Stylistic issues](#stylistic-issues)
- [Substantial issues](#substantial-issues)
- [Suggestions](#suggestions)
- [Questions](#questions)
- [Todo](#todo)
- [Next steps](#next-steps)
- [References](#references)

## Stylistic issues

- There are a number of superficial *stylistic issues* with both of the files (e.g. spacing, variable declaration style, quotation marks). But addressing those is very easy. All you've got to do is *lint* your files. I've added an option to `grunt` that will allow you to run the checks.

    ```bash
    $ grunt jscs
    ````

    The only problem with running that command right now is that your files are in the *bin/*, not in *src/*, so `grunt` will ignore your files. Until the Essay Builder has been integrated into the rest of the system, you can run `jscs` against individual files, so things to do for now:

    1. Install the *npm* package `jscs` *globally*:

       ```bash
       $ npm install -g jscs
       ```

    2. Run jscs against your files individually (the config option *might* be optional):

       ```bash
       $ jscs -c .jscsrc web/static/js/bin/topics/topic-interface.js
       $ jscs -c .jscsrc web/static/js/bin/topics/graph-creator.js
       ```


## Substantial issues

- **Readability**

   - There are *some* comments, but not every major function is documented.

   - Add "end comments" to long functions. See [JS Guide]::[Comments] for suggestions.

- **Maintanability**

   - There are no *unit tests*, so those should be added in the future.

   - There are no high-level, functional, behavioral, integration tests as well. Talk to the QA team.

- **Testability**

   - A first step towards making your code testable is to distinguish between pure vs impure functions. Those that mutate shared state are impure, those that transforms inputs to outputs without modifying external state are pure. That's a rather loose definition of purity, but for our purposes it should suffice. I'll add a note about all this pure vs impure jazz in the guide.

- **Error-handling**

   - Alerts are not a good way of informing users of problems. Look at the main app's use of *Bootstrap Notify* and use that. I'm planning on creating a general-purpose Logger/Errer that will start by using that jquery plugin.

- **Naming**

   - The names of bindings are pretty sensible. What I see not enough of is the use of naming to refer to DOM hooks via jQuery. All DOM hooks should be declared at the top of the modules and must start with the letter '$'. See [JS Guide]::[DRY] for the details.

   - Instead of short and unintuitive (to the outsider) identifiers (e.g. `sou`, `tar`), use more meaningful ones. Especially when those names don't occur too often. Note that this doesn't apply to conventionally short identifiers, such as those used in for loops (e.g. `i`, `j`, `k`).

- **Globals**

   - Modules must not rely on non-built-in globals for reading, and must not pollute the globabl scope with arbitrary random identifiers. Use `Context` and its namespaces to read data from Django's templates and import directly (once integrated you'll be able to use requireJS for this) to read from other modules. See [JS Guide]::[Context] for more on `Context` and its uses.

- **Comments**

   - If you're assuming a variable to contain data of a specific format, make it a little explicit by adding an example in an inline comment that starts with "e.g.". For an example of how to add such examples, take a look at *js/src/models/module.js*.

   - No useless comments. See [JS Guide]::[Comments] for an example.

   - If you're going to include URLs in code, shorten them first.

- **Newlines**

   - Add new lines to separate logically-unrelated lines of code. See [JS Guide]::[Line Breaks].

- **Hoisting**

   - All variables should be lifted to the very tops of their closest parent functions. See [JS Guide]::[Hoisting] for the details.

- **Equality**

   - See [JS Guide]::[Equality].

- **DRY**

   - Extract repeated logic out. See [JS Guide]::[DRY] for some details.


## Suggestions

- In `forEach` (as well as in `map`, `filter`, etc.), passing an `i` to the callback that doesn't make use of the index is unecessary. Pass only what you need.

- Instead of gigantic `switch` statements, use `map[key]` to dispatch on `key`s, so that the data (here `map`) can be extracted out into a *config* file or requested via AJAX and dereferenced. See my `getRelatedWords` for an example of that).

- Instead of `var that = this`, use [[thisArg]] of higher-order functions and `.bind`.

- Instead of `map(...)` use `forEach(...)` if the return value doesn't matter.

- If you're going to add a `return;` statement, check if it's really needed, as that (viz. `undefined`) is the default return value when we 'fall off' the end of the function.

- Adding `return;` to the end of a function doesn't do anything.

- Function names are sometimes misleading. For example, `changeTextOfNode` sounds like a function that mutates something, but it turns out that it also returns a value. It's of course possible to have a mutator that also returns a meaningful value, but such functions are not easy to understand or test.

- High [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity#Definition) makes functions hard to understand, hard to test. Try to keep branching to the minimum.

- I noticed a strange use of `filter`, where besides returning from the callback, you were also mutating state. I don't think in this particular case there is anything buggy going on, but doing so is a very dangerous thing. It can lead to a problem known as [iterator invalidation](http://stackoverflow.com/q/16904454/2672370), which usually happens when one mutates the collection one is iterating over.

- Instead of `filter`, it's often more readable to use Underscore's `_.where` and `_.findWhere` utilities.

- C-style `for` loops are problematic (hoisting, off-by-one, etc), so use `map`, `forEach`, etc. whenever possible.

- Use `typeof source !== undefined` instead of `source !== undefined`, as in some execution environments, `undefined` can be overwritten.

- Instead of `obj.key = null` do `delete obj.key`.

- Replace:

   ```js
   if (Math.sqrt(x * x + y * y) <= r2) {
       return true;
   }
   return false;
   ```
   with:
   ```js
   return Math.sqrt(x * x + y * y) <= r2
   ```

- Replace:

   ```js
   var result = true;
   if (y_length < position) {
       return false;
   }
   return result;
   ```
   with:
   ```js
   return (y_length >= position)
   ```

- Replace:

   ```js
   var source = thisGraph.nodes.filter(function(n) {
       return n.title == sentence[0];
   })[0];
   ```
   with:
   ```js
   var source = _.findWhere(thisGraph.nodes, {title: sentence[0]});
   ```

- Replace:

   ```js
   if (d.id == parseInt(getSourceNodeId())) {
       return 'conceptg selected';
   }
   if (d.id == parseInt(getTargetNodeId())) {
       return 'conceptg target';
   }
   return 'conceptg';
   ```
   with (notice the strict equality change):
   ```js
   return 'conceptg' +
       (d.id === parseInt(getSourceNodeId()) ? ' selected' : '') +
       (d.id === parseInt(getTargetNodeId()) ? ' target': '');
   ```

- Replace:

   ```js
   var result;
   switch (topic) {
       case 'music':
           result = ['Adagio', 'Band', ...];
           break;
       case 'Adagio':
           result = ['word1', 'word2', ...];
           break;
       case 'word1':
           result = ['word4', 'word5', ...];
           break;
       case 'word2':
           result = ['word10', 'word11', ...];
           break;
       case 'word3':
           result = ['word10', 'word11', ...];
           break;
       default:
           result = ['word1', 'word2', ...];
   }
   return result;
   ```
   with:
   ```js
   var assoc = {
       music: ['Adagio', 'Band', ...],
       Adagio: ['word1', 'word2', ...],
       word1: ['word4', 'word5', ...],
       word2: ['word10', 'word11', ...],
       word3: ['word10', 'word11', ...],
       _: ['word1', 'word2', ...],
    };
    return assoc[topic] || assoc['_'];
   ```

- Replace:

   ```js
   thisGraph.circles.filter(function(dval) {return dval.id === d.id;})
   ```
   with:
   ```js
   _.where(thisGraph.circles, {id: d.id})
   ```

## Questions

1. Could those 'transform', 'scale', etc. (from, say `zoomWithSliderLibrary`) be extracted out into functions for interpolating? They look a bit messy and tightly-coupled.

## Todo

- Augusto:

   1. Please answer the [Questions](#questions) above. Email answers to my WriteLab email.

   2. Follow the directions for liting your files under [General](#general). Once [PR 167] is merged into develop, get the latest *.jscsrc* file and run the linter again (may need to run `npm update` to make sure you have the right dependencies). As I mentioned, once the Essay Builder is integrated into the rest of the system, you'll simply run `grunt jscss` to do the linting.

   3. Incorporate Hunan's suggestions listed in this document.

- Hunan:

   1. ~~Add "requireTrailingComma" and "disallowMultipleVarDecl" (strict) to *.jscsrc*.~~

   2. Add a note about *pure vs impure* as they relate to testability.

## Next steps

- At our meeting tomorrow we can go over this document.
- The next major step is integrating the files into the rest of the system. This will involve lots of modularization and decoupling.
- Another major step is to add some useful unit and integration tests. This should follow the integration because a modularized app will be easier to unit test, because there will be lots of little isolated units. Also, you'll see that there is something I call a *director*, which encapsulates the logic of the app, and testing the director is one way of testing for integration. We'll go over the details in due time.

## References

[JS Guide] [JavaScript Substance Guide](https://github.com/hunan-rostomyan/codeguides/blob/93a061220e4413450001f60c1cfa7e36b6caa10c/Dev/Frontend/JavaScript/Substance%20Guide.md)

[PR 151]: https://bitbucket.org/writetrack/writetrack/pull-requests/151/topic-modeler/diff
[PR 167]: https://bitbucket.org/writetrack/writetrack/pull-requests/167/1-disallow-all-multiple-var-declarations-2/diff
[JS Guide]: https://github.com/hunan-rostomyan/codeguides/blob/93a061220e4413450001f60c1cfa7e36b6caa10c/Dev/Frontend/JavaScript/Substance%20Guide.md
[Comments]: https://github.com/hunan-rostomyan/codeguides/blob/master/Dev/Frontend/JavaScript/Substance%20Guide.md#comments
[DRY]: https://github.com/hunan-rostomyan/codeguides/blob/master/Dev/Frontend/JavaScript/Substance%20Guide.md#dry
[Equality]: https://github.com/hunan-rostomyan/codeguides/blob/master/Dev/Frontend/JavaScript/Substance%20Guide.md#dry
[Hoisting]: https://github.com/hunan-rostomyan/codeguides/blob/master/Dev/Frontend/JavaScript/Substance%20Guide.md#hoisting
[Line Breaks]: https://github.com/hunan-rostomyan/codeguides/blob/master/Dev/Frontend/JavaScript/Substance%20Guide.md#line-breaks
[Context]: https://github.com/hunan-rostomyan/codeguides/blob/master/Dev/Frontend/JavaScript/Substance%20Guide.md#context
