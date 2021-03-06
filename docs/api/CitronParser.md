---
title: "CitronParser"
permalink: /parsing-interface/api/CitronParser/
layout: default

---

[Citron] > [Parsing interface] > [`CitronParser`]

[Citron]: /citron/
[Parsing interface]: /citron/parsing-interface/
[`CitronParser`]: .

# CitronParser

_Protocol_

Defines the Citron parser interface.

The parser code generated by Citron defines a class that conforms to this
protocol.

  - [Parsing](#parsing)
      - [`consume(token: CitronToken, tokenCode: CitronTokenCode)`](#consumetoken-citrontoken-tokencode-citrontokencode)
      - [`endParsing()`](#endparsing)
      - [`consume(lexerError: Error)`](#consumelexererror-error)
  - [Errors](#errors)
      - [`UnexpectedTokenError`](#unexpectedtokenerror)
      - [`UnexpectedEndOfInputError`](#unexpectedendofinputerror)
      - [`StackOverflowError`](#stackoverflowerror)
  - [Stack size](#stack-size)
      - [`maxStackSize: Int?`](#maxstacksize-int)
      - [`maxAttainedStackSize: Int`](#maxattainedstacksize-int)
  - [Tracing](#tracing)
      - [`isTracingEnabled: Bool`](#istracingenabled-bool)
      - [`isTracingPrintsSymbolValues: Bool`](#istracingprintssymbolvalues-bool)
      - [`isTracingPrintsTokenValues: Bool`](#istracingprintstokenvalues-bool)
  - [Error capturing](#errorcapturing)
      - [`errorCaptureDelegate: CitronErrorCaptureDelegate?`](#errorcapturedelegate-citronerrorcapturedelegate)
      - [`consume(lexerError: Error)`](#consumelexererror-error)
  - [Associated Types](#associated-types)
      - [`CitronToken`](#citrontoken)
      - [`CitronTokenCode`](#citrontokencode)
      - [`CitronNonTerminalCode`](#citronnonterminalcode)
      - [`CitronSymbolCode`](#citronsymbolcode)
      - [`CitronResult`](#citronresult)
      - [`CitronErrorCaptureDelegate`](#citronerrorcapturedelegate)
      - [`CitronErrorCaptureState`](#citronerrorcapturestate)

---

## Parsing

### `consume(token: `[`CitronToken`]`, tokenCode: `[`CitronTokenCode`]`)`

> Consumes one token.
>
> This should be called multiple times to pass a sequence of tokens to
> the parser. Typically, a separate tokenization stage would generate
> this sequence of tokens.
>
> **Parameters:**
>
>   - `token`
>
>     The semantic value of the token, as seen by the code blocks in the
>     grammar.
>
>     The type of this argument is the
>     [%token_type](/citron/grammar-file/#token_type) type specified in
>     the grammar file, available to the parser class as the associated
>     type [`CitronToken`].
>
>   - `code`
>
>     The token code that Citron knows this token by.
>
>     The Citron-generated parser class contains an enum called
>     [`CitronTokenCode`] that lists all the terminals in the grammar.
>     This argument should be a value of that enum.
>
> **Return value:**
>
>   - None
>
> **Throws:**
>
>   - May throw one of these errors:
>       - [`UnexpectedTokenError`](#unexpectedtokenerror)
>       - [`StackOverflowError`](#stackoverflowerror)
>
>   - Any errors thrown in the [code blocks] of rules may propagate up
>     to the caller of this method.

### `endParsing()`

> Signifies the end of input. This should be called when there are no
> more tokens to consume.
>
> **Parameters:**
>
>   - None
>
> **Return value:**
>
>   - Returns the parse result.
>
>     The parse result is the value returned by the code block of the
>     [start rule] of the grammar.
>
>     The return type of this method is the [semantic type] of the
>     [start symbol], available to the parser class as the associated
>     type [`CitronResult`].
>
> **Throws:**
>
>   - May throw one of these errors:
>       - [`UnexpectedEndOfInputError`](#unexpectedendofinputerror)
>       - [`StackOverflowError`](#stackoverflowerror)
>
>   - Any errors thrown in the [code blocks] of rules may propagate up
>     to the caller of this method.

[start rule]: /citron/grammar-file/#start-rule
[start symbol]: /citron/grammar-file/#start_symbol
[semantic type]: /citron/grammar-file/#types-for-non-terminals
[code blocks]: ../grammar-file/#code-blocks

## Errors

### `UnexpectedTokenError`

> A token was encountered at a point that is not supported by the
> specified grammar.

### `UnexpectedEndOfInputError`

> End of input was encountered before the grammar could be satisfied.

### `StackOverflowError`

> Signifies that parsing was aborted because to continue parsing the
> number of entries in the stack needs to exceed the [`maxStackSize`] set.
>
> If [`maxStackSize`] is `nil` (the default), this error is never thrown,
> and the parser stack is allowed to grow without overflowing.

## Stack size

### `maxStackSize: Int?`

> If this is `nil` (the default), there is no restriction on how many
> entries the parser stack can have.
>
> If this is `non-nil`, this specifies the maximum number of entries
> that the parser's stack can be allowed to hold. If an input requires
> the stack to grow above that, it would cause a [`StackOverflowError`]
> to be thrown, and parsing shall be aborted.

### `maxAttainedStackSize: Int`

> This can be queried at the end of the parse to see the maximum number of
> entries that the stack ever held during the parse.
>
> We can, for example, observe the effect of left-recursive vs
> right-recursive rules on how the stack grows.

[`StackOverflowError`]: #stackoverflowerror
[`maxStackSize`]: #maxstacksize-int

## Tracing

### `isTracingEnabled: Bool`

> If this is set to `true`, information on how the parsing happens is
> printed out. This can be used to debug the parser.
>
> The default value is `false`.

### `isTracingPrintsSymbolValues: Bool`

> If this is set to `false` (the default), tracing prints only the
> symbol codes and not the semantic values of the symbols.
>
> If this is set to `true`, tracing also prints semantic values of the
> symbols using the `String(describing:)` string initializer.
>
> The default value is `false`.

### `isTracingPrintsTokenValues: Bool`

> If this is set to `false` (the default), tracing prints only the token
> codes and not the semantic values of the tokens.
>
> If this is set to `true`, tracing also prints semantic values of the
> tokens using the `String(describing:)` string initializer.
>
> The default value is `false`.

## Error capturing

### `errorCaptureDelegate: `[`CitronErrorCaptureDelegate`]`?`

> To activate error capturing, this should be set to a class conforming
> to the protocol referred to by the [`CitronErrorCaptureDelegate`]
> associated type.
>
> If this is `nil`, error capturing is not activated even if the
> [grammar file] contains [%capture_errors] directives.
>
> The default value is `nil`.

### `consume(lexerError: Error)`

> To enable error capturing of lexer errors, the lexer errors can be
> passed to the parser using this method. Each lexer error should be
> passed to this method, one at a time, as and when it is found during
> tokenization.
>
> **Parameters:**
>
>   - `lexerError`
>
>     The lexer error that needs to be error-captured.
>
> **Return value:**
>
>   - None
>
> **Throws:**
>
> If the error cannot be saved for capturing, this method throws the
> error passed in as `lexerError`.
>
> This can happen in one of these scenarios:
>
>  1. If there are no [error capturing nonterminals][%capture_errors]
>     currently being parsed, _or_
>  2. The [`errorCaptureDelegate`]'s
>     [`shouldSaveErrorForCapturing(error:)`] method returns `false`

[`errorCaptureDelegate`]: #errorcapturedelegate-citronerrorcapturedelegate
[%capture_errors]: /citron/grammar-file/#capture_errors
[grammar file]: /citron/grammar-file
[error capturing]: /citron/error-capturing/
[`shouldSaveErrorForCapturing(error:)`]: /citron/parsing-interface/api/CitronErrorCaptureDelegate/#shouldsaveerrorforcapturingerror-error

## Associated Types

These associated types are bound to types defined by the
Citron-generated parser code.

### `CitronToken`

> The semantic type of each token, as seen by the code blocks in the
> grammar.  This is the type defined by
> [%token_type](/citron/grammar-file/#token_type) type in the grammar
> file.

### `CitronTokenCode`

> This is an enum of all tokens (or terminals) used in the grammar file.
>
> If a [%tokencode_prefix](/citron/grammar-file/#tokencode_prefix) is
> specified, the enum values will use that prefix.

### `CitronNonTerminalCode`

> This is an enum of all non-terminals used in the grammar file.
>
> If a [%nonterminalcode_prefix](/citron/grammar-file/#nonterminalcode_prefix) is
> specified, the enum values will use that prefix.

### `CitronSymbolCode`

> This is an enum that represents all symbols (both terminals and
> non-terminals) used in the grammar file.

### `CitronResult`

> The semantic type of the [start
> symbol](/citron/grammar-file/#start_symbol) as specified in the
> grammar file.

### `CitronErrorCaptureDelegate`

> This is typealiased to a protocol that the [`errorCaptureDelegate`]
> should conform to. The name and contents of the protocol are
> automatically generated from the [grammar file].
>
> See the [`CitronErrorCaptureDelegate`](../CitronErrorCaptureDelegate)
> page for more information.

### `CitronErrorCaptureState`

> This is typealiased to a structure populated by Citron to pass the
> error capturing state to
> [`CitronErrorCaptureDelegate`](#citronerrorcapturedelegate) methods.
>
> See the [`CitronErrorCaptureState`](../CitronErrorCaptureState) page
> for more information.

[`CitronToken`]: #citrontoken
[`CitronTokenCode`]: #citrontokencode
[`CitronResult`]: #citronresult
[`CitronErrorCaptureDelegate`]: #citronerrorcapturedelegate
[`CitronErrorCaptureState`]: #citronerrorcapturestate
