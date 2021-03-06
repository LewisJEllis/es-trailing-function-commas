# Proposal to allow trailing commas in function parameter lists

In some codebases/style guides there are scenarios that arise where function calls and definitions are split across multiple lines in the style of:

```js
 1: function clownPuppiesEverywhere(
 2:   param1,
 3:   param2
 4: ) { /* ... */ }
 5: 
 6: clownPuppiesEverywhere(
 7:   'foo',
 8:   'bar'
 9: );
```

In these cases, when some other code contributer comes along and adds another parameter to one of these parameter lists, they must make two line updates:

```js
 1: function clownPuppiesEverywhere(
 2:   param1,
 3:   param2, // updated to add a comma
 4:   param3  // updated to add new parameter
 5: ) { /* ... */ }
 6: 
 7: clownPuppiesEverywhere(
 8:   'foo',
 9:   'bar', // updated to add a comma
10:   'baz'  // updated to add new parameter
11: );
```

In the process of doing this change on code managed by a version control system (git, subversion, mercurial, etc), the blame/annotation code history information for lines 3 and 9 get updated to point at the person who added the comma (rather than the person who originally added the parameter).

To help mitigate this problem, some other languages (Python, D, Hack, ...probably others...) have added grammar support to allow a trailing comma in these parameter lists. This allows code contributors to always end a parameter addition with a trialing comma in one of these per-line parameter lists and never have to worry about the code attribution problem again:

```js
 1: function clownPuppiesEverywhere(
 2:   param1,
 3:   param2, // Next parameter that's added only has to add a new line, not modify this line
 5: ) { /* ... */ }
 6: 
 7: clownPuppiesEverywhere(
 8:   'foo',
 9:   'bar', // Next parameter that's added only has to add a new line, not modify this line
11: );
```

Note that this proposal is exclusively about grammar and makes no changes to semantics, therefore the presence of a trailing comma has no effect on things like `<<function>>.length`.

This repo contains the proposal slides, a version of esprima hacked to allow trailing commas in parameter lists, and a very simple CLI utility to show that it's possible (and easy) to transpile trailing commas to ES5-compatible non-trailing commas in a build step.

For the CLI, you can either give it a single filename argument to read from disk, or you can pipe source text in to it.

## Spec Text

##### [14.1 Function Declarations](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-function-definitions)

[...]

_FormalParameters_ :<br />

&nbsp;&nbsp;[empty]<br />
&nbsp;&nbsp;**_FunctionRestParameter_**<br />
&nbsp;&nbsp;FormalParameterList<br />
&nbsp;&nbsp;**_FormalParameterList_ ,**<br />
&nbsp;&nbsp;**_FormalParameterList_ , _FunctionRestParameter_**<br />

_FormalParameterList_ :<br />

&nbsp;&nbsp;**~~_FunctionRestParameter_~~**<br />
&nbsp;&nbsp;**~~_FormalsList_~~**<br />
&nbsp;&nbsp;**~~_FormalsList_ , _FunctionRestParameter_~~**<br />
&nbsp;&nbsp;**_FormalParameter_**<br />
&nbsp;&nbsp;**_FormalParameterList_ , _FormalParameter_**<br />

**~~_FormalsList_~~** :<br />

&nbsp;&nbsp;**~~_FormalParameter_~~**<br />
&nbsp;&nbsp;**~~_FormalsList_ , _FormalParameter_~~**<br />

##### [14.1.3 Static Semantics: BoundNames](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-function-definitions-static-semantics-boundnames)

_FormalParameters_ : [empty]<br />

1. Return an empty List.

_FormalParameters_ : _FormalParameterList_, _FunctionRestParameter_ <br />

1. Let _names_ be BoundNames of _FormalParameterList_.
2. Append to _names_ the BoundNames of _FunctionRestParameter_.
3. Return _names_.

_FormalParameters_ : _FormalParameterList_, _FormalParameter_ <br />

1. Let _names_ be BoundNames of _FormalParameterList_.
2. Append to _names_ the BoundNames of _FormalParameter_.
3. Return _names_.

##### [14.1.5 Static Semantics: ContainsExpression](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-function-definitions-static-semantics-containsexpression)
 
 [...TODO..interuppted, so committing intermediate...]
 
##### [A.2 Expressions](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-expressions)

_CallExpression_ :<br />

&nbsp;&nbsp;_MemberExpression_ _Arguments_<br />
&nbsp;&nbsp;_SuperCall_<br />
&nbsp;&nbsp;_CallExpression_ _Arguments_<br />
&nbsp;&nbsp;_CallExpression_ [ _Expression_ ]<br />
&nbsp;&nbsp;_CallExpression_ . _IdentifierName_<br />
&nbsp;&nbsp;_CallExpression_ _TemplateLiteral_<br />

_Arguments_ :<br />

&nbsp;&nbsp;( )<br />
&nbsp;&nbsp;**( ... _AssignmentExpression_ )**<br />
&nbsp;&nbsp;( _ArgumentList_ )<br />
&nbsp;&nbsp;**( _ArgumentList_ , )**<br />
&nbsp;&nbsp;**( _ArgumentList_ , ... _AssignmentExpression_ )**<br />

_ArgumentList_ :<br />

&nbsp;&nbsp;_AssignmentExpression_<br />
&nbsp;&nbsp;**~~... _AssignmentExpression_~~**<br />
&nbsp;&nbsp;_ArgumentList_ , _AssignmentExpression_<br />
&nbsp;&nbsp;**~~_ArgumentList_ , ... _AssignmentExpression_~~**<br />
