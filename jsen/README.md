# JSEN

> JSEN is a Dynamic Data Interchange Format based on JavaScript expressions.

Far beyond what a regular static Data Interchange Format like JSON permits, JSEN allows us to convey math and logical expressions, conditional expressions, function constructs and lots of other JavaScript expressions. And it comes as a parser combinator for easy extensibilty into some other derived Domain Specific Language \(DSL\).

The power of JSEN expressions lies in their being evaluable. To put this power in your hands, the static `Jsen.parse()` function returns a expression instance that can be evaluated multiple times with different contexts for context-based results. This is done with the `eval()` instance method that works like the `eval()` function in many prorgamming languages, although these have their peculiar costs implications. The cycle completes with the `toString()` instance method that returns us the string representation of the expression instance.

The new possibilities with JSEN are endless. For example, this paves the way for accepting logical expressions from configuration files and API definition files.

[Get Started](guide.md)

