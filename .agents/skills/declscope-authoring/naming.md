# Naming under declscope

This file is part of the [declscope-authoring](SKILL.md) skill. It applies only when the naming rule is on: `rules.naming.qualify` set to `always`, or to `ondemand` in a package with a second namespace. The namespace may sit anywhere in the name, but it must start a word. Its right edge may fall inside a word: `loadUserID` carries `user`, and `poweruserID` does not.

Fields, interface methods, and methods written in their type's file are exempt. A method in another file must carry that file's namespace. So do helpers in a test file. `TestXxx`, `BenchmarkXxx`, `FuzzXxx` and `ExampleXxx` in a `_test.go` file are exempt, since `go test` finds them by name.

**Prefer natural word order.** With a verb namespace, a prefix reads as a command: `collectAddFunc` is an instruction, and `addFuncToCollection` is a name. An entry point is the exception, since `collectFiles` already says what it does.

**A verb prefix can promise a side effect the code does not have.** A name that starts with a verb reads as that action. In `delete.go`, a function that only checks whether a user may be deleted must not be called `deleteUser`. A reader expects it to delete the user. Name it for what it does, and fit the namespace in elsewhere: `canDeleteUser`, `usersToDelete`. Keep the verb first only for code that performs the action.

A verb file name is still fine. Its regular noun and past forms often carry it already, since the namespace's right edge may fall inside a word. In `collect.go`, `collectedFiles`, `collectionKeys` and `addToCollection` all carry `collect`. A form that changes the verb's spelling, such as `deletion` for `delete`, needs another fit or a vocabulary entry. The two generated forms are the exception. See [When the name spells the namespace in another form](#when-the-name-spells-the-namespace-in-another-form).

## Fitting the namespace in

A prefix is one fit, and often the right one: `userProfile` is a prefix. Do not use it by reflex, though. Choose the place where the namespace reads as part of the phrase:

| Fit it in as | Namespace | Examples |
| --- | --- | --- |
| A modifier before a noun | `user` | `userProfile`, `userRow` |
| The object after a verb | `user` | `loadUser`, `deleteUserByID` |
| A phrase after the main word | `collect` | `addFuncToCollection` |
| An inflected form | `collect`, `store` | `collectedFiles`, `storingKeys`, `storedKeys` |
| A word it begins, ending inside it | `parse` | `SpecifierParser` |

**Then read the name aloud.** A command (`collectAddFunc`), a stack of nouns with no grammar, or a word used against its meaning means another fit is needed. When no fit reads naturally, the name is not the problem. Suspect the file instead: see [An ill-fitting prefix means the declaration is in the wrong file](#an-ill-fitting-prefix-means-the-declaration-is-in-the-wrong-file).

## Carrying the namespace does not make a name accurate

The rule checks that the namespace is in the name. It cannot check that the rest of the name is true. Name the declaration for exactly what it is, then make sure the namespace is in it. Before naming a collection, read the code that fills it.

| Written | Wrong because | Better |
| --- | --- | --- |
| `methodNames`, filled with every function | Plain functions are in it too, not only methods | `funcNames`, or fill it with methods only |
| `collectInfo` | The namespace is the only real word, so the name says nothing | Name what it holds: `collectedFiles` |
| `orderSortKeys` in `order.go`, read by every file | The prefix claims one owner | Move it. See the next section |

When the accurate name reads badly with the namespace in it, do not bend the words. Read it as a signal about placement.

## An ill-fitting prefix means the declaration is in the wrong file

Sometimes the namespace a name must carry does not describe the thing. Then the declaration sits in the wrong file, and renaming it hides that.

For example, a tax rate table that `invoice.go` and `refund.go` both read was placed in `invoice.go`. So it had to be called `invoiceTaxRates`, and `refund.go` read `invoiceTaxRates` as if it borrowed something of the invoice's. It belongs in `tax.go`, where it is just `taxRates`.

| The prefix you must add | Do this |
| --- | --- |
| Describes what the thing is | Keep it |
| Belongs to one caller, but several parts use the thing | Move it to a file named for its concept, or to the shared-model file. A concept file often carries the name for free: `taxRates` carries `tax` |
| Reads badly for most names in the file | Often the file name is wrong. See [Reading a saturation](../declscope-adoption/SKILL.md#reading-a-saturation) |

This reasoning holds in a package laid out for declscope. In a feature change to a package whose findings are baselined, the file name may follow the package's own convention instead. On a new file, keep that name and set the namespace. On an existing file, set one when you can finish what it brings back. [Adding a feature](SKILL.md#adding-a-feature) explains both.

**Do not create a file just to satisfy a name.** A new twenty-line file holding one declaration that only its old file uses is a warning sign. Rename the declaration where it was. A new file for new code, named in the package's convention, is another matter. A declaration that several parts share is different: move it, as the table above says.

## When the name spells the namespace in another form

Only the namespace's own spelling and two generated forms carry it: a final `e` dropped before `ing` (`store` → `storing`) and `y` turned to `i` (`apply` → `applies`, `applied`). Generation only makes the namespace longer. Nothing else is guessed:

| File | Name | Carried | Why |
| --- | --- | --- | --- |
| `store.go` | `storingKeys` | Yes | A generated form |
| `tracing.go` | `traceValue`, `tracerType` | No | The stem of an inflected file name is not derived |
| `walking.go` | `walkSpeed` | No | Same |
| `index.go` | `indicesSorted` | No | An irregular form |

**Prefer the base form of a word for a new file name.** It is a recommendation, not a requirement. `trace.go` is carried by `traceValue`, `tracerType` and `tracingStart`. `tracing.go` is carried only by the last. Generated forms and the free right edge both run from the namespace to longer words, so the base form reaches every form the inflected file name reaches, and more. The same holds for `entry.go` over `entries.go`, `walk.go` over `walking.go`, and `user.go` over `users.go`. An inflected file name is allowed. It only fits declscope worse, so choose it when it reads clearly better.

For an existing file, renaming it to the base form keeps every name that carried the old namespace. It still changes the namespace, so it is not free. Rename its test files too, and update any `//declscope:namespace` that names it and each `// reason` that names the file. Its baselined findings report again. Fix them, as [Refactoring a part of the package](SKILL.md#refactoring-a-part-of-the-package) explains. Do not rename an existing file only for this. Mention the rename as an option, unless the file is in a refactoring the user asked for. When it is too wide, or the owner prefers the current name, a vocabulary entry is the smaller change.

`rules.naming.vocabulary` lists extra words that carry a namespace. A listed word goes through the same test as the namespace, so `trace` also covers `tracer`. When the namespace looks inflected, the diagnostic names the key: `list that form under rules.naming.vocabulary.tracing`.

```yaml
rules:
  naming:
    vocabulary:
      tracing: [trace]
      index: [indices]
```

Use it for a word that names what the namespace names: another form of it, or the domain's word for part of it (`mouse: [wheel]`). For those, the rename the message offers would stutter (`tracingTraceValue`).

| The name | Do this |
| --- | --- |
| Spells the base form of an inflected file name | Propose a vocabulary entry, and mention renaming the file to the base form as an option |
| Spells the namespace in another form the base form cannot reach (`indices`), or the domain's word for part of it | Propose a vocabulary entry |
| Follows a Go idiom that never spells the file's name, such as `WithTimeout` in `options.go` | Propose a vocabulary entry: `options: [with]` |
| Spells a different word, which describes the thing better | The file may be named wrong, or the declaration may be in the wrong file. See [An ill-fitting prefix](#an-ill-fitting-prefix-means-the-declaration-is-in-the-wrong-file) |
| Would need a long list of words | Split the file. A long list means the file declares things it is not about |

A vocabulary entry is a config change, so ask first. List the word the name actually spells. Do not list a stem you guessed: `trac` would also carry `track`.

## In a package that has a baseline

**When the naming rule reports that a new name cannot carry the file's namespace**, such as `parseAttribute` in `parser.go`, fix the file's namespace: `//declscope:namespace parse`. Do it when you can fix the file's recorded findings that come back, as [Refactoring a part of the package](SKILL.md#refactoring-a-part-of-the-package) does. Otherwise put the new code in a new file named in the package's convention, such as `parser_function_call.go`, with its own namespace. Propose the file's namespace in your report.

A vocabulary entry fits a word that names what the namespace names in another spelling, as [When the name spells the namespace in another form](#when-the-name-spells-the-namespace-in-another-form) explains. It reaches every package under the config. When fixing the file's namespace would let the names carry it, such as `parse` for `parser`, prefer that.
