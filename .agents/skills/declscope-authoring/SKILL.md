---
name: declscope-authoring
description: "Write or change Go code in a repository that runs declscope, which shows as a .declscope.y*ml or baseline file, //declscope: comments, or declscope in CI. Read this before adding, naming or moving a declaration, a helper that several files use, or a test. Read it too before splitting a file, writing a //declscope: directive, or acting on a declscope diagnostic. Covers where code belongs, naming it accurately, and fixes that hide a problem."
license: MIT
x-embedded-by: declscope
x-embedded-version: 0.24.2
x-embedded-at: "2026-11-05T22:26:52Z"
x-embedded-digest: "sha256:43e21045fe8b3065a93166e19e8c406ef64c9777046e873c5fd8bbb531e303d6"
---

# Writing code under declscope

Written against **declscope 0.24.2**. Check the version first with `declscope -V=full`: this describes how that release behaves, not how an older one does.

This skill is for everyday work in a repository that already runs declscope. Introducing it, choosing its configuration, and clearing the whole repository's baseline are in [declscope-adoption](../declscope-adoption/SKILL.md), installed beside this one. Changing one package whose findings are baselined, including clearing its entries, is in [Changing a package that has a baseline](#changing-a-package-that-has-a-baseline). What each directive and config key means is in [the README](https://github.com/mpyw/declscope#readme).

## The model

A file is a namespace. Its name comes from the file name: `user_repository.go` is `userRepository`. A `//declscope:namespace` above the package clause names it instead.

| Declaration | Scope by default | Who may use it |
| --- | --- | --- |
| Unexported | `private` | Its own namespace only |
| Exported | `shared` | Any file of the package |
| With `//declscope:shared`, or in a file with one above the package clause | `shared` | Any file of the package |
| With `//declscope:private`, or in a file with one above the package clause | `private` | Its own namespace only |

**A field or an interface method takes the nearest scope stated for it.** The order is its own directive, its type's, a file-level one, then the default. With nothing stated, an unexported field of an exported type is private.

A method is not a member. It belongs to the file it is written in, and a directive on its type does not reach it.

Which rules are on, and which config governs a package, is in [Check what is switched on](../declscope-adoption/SKILL.md#check-what-is-switched-on). Read "Checks in force" from `declscope survey ./...` before trusting a count.

**A count of zero may mean nothing is checked.** The naming rule is off unless a config turns it on. Under `qualify: ondemand` it applies once a package has a second namespace. It skips exported names unless `rules.naming.exported` is on.

## Where a new declaration goes

**Put it in the file whose concern it is.** That choice decides its namespace, so it decides who may use it and what its name must carry. The directive and the name follow from it.

| The declaration is used by | Put it | Scope |
| --- | --- | --- |
| Code in one file | That file | Default. No directive |
| Several files, as a shared helper on purpose | A file named for its concept, not the file of its first caller | `//declscope:shared // <who uses it and why>`. A file-level one only when the file is shared by nature. See [Directives](#directives) |
| Most files of the package, as its shared model | A core file: `//declscope:core` plus a file-level `//declscope:shared` | Package-wide, from the file directive |
| Another package | Wherever it belongs, exported | Default for exported names |

**A core file does not share its declarations by itself.** `//declscope:core` sets only the namespace, and exempts the file from the naming rule. Its unexported declarations are still private to the core. A core file whose declarations are shared also needs a file-level `//declscope:shared`.

**Before widening, ask whether the code is in the wrong file.** A boundary report on your new use says a file reached into another namespace. Often the function you are writing belongs beside what it uses. When the declaration is shared on purpose, check that it sits in a file named for its concept before you write the directive. When you add a feature to a package whose findings are baselined, keep to its layout instead. See [Changing a package that has a baseline](#changing-a-package-that-has-a-baseline).

**Do not create a file just to satisfy a name.** A new twenty-line file holding one declaration that only its old file uses is a warning sign. Rename the declaration where it was. A declaration that several parts share is different: move it, as the table above says.

## Changing a package that has a baseline

A baseline is a file, usually `.declscope-baseline.yaml`, that records the findings a repository had when it adopted declscope. declscope does not report what it records. Only new findings fail the check: `declscope` and `declscope shrink`, as CI runs them.

This section is for changing a package whose findings are recorded in a baseline. Such a package was laid out before declscope. A baseline file alone does not say that: it covers a whole repository, and a package laid out for declscope may sit beside the recorded ones. To check a package, run `declscope survey ./path/to/pkg` and read its `baselined` column. A number above zero means it applies. Searching the baseline file for the package's import path also works.

In this skill, the owner is the user you are working for. Handle a proposal to the owner this way:

| The situation | Do this |
| --- | --- |
| Your change cannot pass the check without an answer | Ask in the chat, and wait. Do not finish with a failing check, a bent name or an ignore. When nobody can answer, as in a non-interactive run, leave that part undone. Finish the rest only when it works without it. When it does not, change nothing. Put the question in your report |
| The owner declines | Ask the owner to choose between two, and wait as above. One is a new file named in the package's convention, with its own `//declscope:namespace`. In a package that keeps a layer in one file, name it after that file, such as `parser_function_call.go`. Name the new file and its namespace in the question. A file the owner chose is not one made just to satisfy a name. The other is `//declscope:ignore qualify // <reason>` on the name. `qualify` is the naming rule's name. An ignore the owner chose is a decision |
| Your change passes without it | Write it in your report, and finish |

What you do depends on what you were asked:

| The request | Do this |
| --- | --- |
| Add a feature or fix a bug, and nothing more | Keep the package's layout, as [Adding a feature](#adding-a-feature) explains |
| Refactor some files, a part of the package, or the whole package, with or without a feature | Inside that scope, follow this skill as in a package laid out for declscope. Outside it, keep the layout. See [Refactoring a part of the package](#refactoring-a-part-of-the-package) |
| Introduce declscope, or clear the baseline of the whole repository | Follow [declscope-adoption](../declscope-adoption/SKILL.md). Its order of work deletes the baseline file, which covers every package |

The refactoring needs a clear request from the user, such as "clean up `parser.go` while you are there" or "refactor this package". A declscope finding that a rename would silence is not one. Nor is a request to fix the reported findings, such as "fix the lint errors". When the request leaves the scope unclear, such as "tidy this up", ask which files it covers. When the user asks to refactor the whole package, the package is the scope. Fix the findings that report again. A declaration whose key did not change keeps its entry, and you mention it. A declaration whose file got a new namespace is not one of them. A request to clear one package's baseline is a refactoring of the whole package that fixes every finding of it. Its entries in the baseline file list those findings. It is done when `-merge` removes all of them. In these cases, do not delete the baseline file. When `-merge` leaves it empty, delete it.

### Adding a feature

A bug fix follows the same rules as a feature. In this skill, a feature change means either.

The package's files may follow another plan. Some packages keep a layer in one file, such as every parse function in `parser.go`. Others follow a naming convention, such as an `ast_` prefix on each type's file. **Add new code the way the package already does it.** Reorganizing the package to suit the rules is a separate change. Propose it, and do not fold it into a feature unless the user asks. When your change happens to remove a baselined finding, leave its entry. An entry that matches nothing is harmless. Mention it in your report. [A baseline is for arriving, not for staying](../declscope-adoption/SKILL.md#a-baseline-is-for-arriving-not-for-staying) covers that work.

| The package | Do this |
| --- | --- |
| Names its files with a shared prefix, such as `ast_object_cons_expr.go`, and you add a new concept | Name the new file the same way. Set its namespace with `//declscope:namespace`, so the names carry it without the prefix |
| Keeps one layer in one file, such as every parse function in `parser.go` | Put the new functions in that file. A method goes in its type's file, as [Common changes](#common-changes) says. The new names carry the file's namespace, as for any file. When no natural name carries the namespace, propose a namespace for the file, as [Fixing an existing file's namespace](#fixing-an-existing-files-namespace) explains |
| Already has a file for the concept you add | Put the new code there, even when that file's existing declarations are baselined. Its names carry that file's namespace. When no natural name does, propose a namespace for the file, as in the row above |

In every row, the new code may use a private declaration of another file. When declscope reports that use, write `//declscope:shared // <who uses it>` on that declaration, such as `// parser.go builds these`. Leave the declaration where it is. When it states its own `//declscope:private`, ask the owner instead.

Here is a new file for the first row. `parser.go` calls `newFunctionCallExpr`. Its test file, `ast_function_call_expr_test.go`, takes the same directive.

```go
// ast_function_call_expr.go
//declscope:namespace functionCallExpr
package hclwrite

// FunctionCallExpr carries functionCallExpr, and the file keeps its name.
type FunctionCallExpr struct{ /* ... */ }

//declscope:shared // parser.go builds these
func newFunctionCallExpr( /* ... */ ) *FunctionCallExpr { /* ... */ }
```

**A namespace directive on an existing file changes its baseline keys.** A baseline entry names the declaration's namespace and the declaration's name. Renaming an existing declaration changes its key in the same way, so in a feature change, leave baselined names as they are. So `//declscope:namespace parse` on an existing `parser.go` makes every baselined finding of that file new again. Adding one to an existing file is therefore a refactoring of that file, and its findings must be fixed. On a file you create, there is no such cost.

#### Fixing an existing file's namespace

Sometimes a new name cannot carry an existing file's namespace. `parseAttribute` in `parser.go` does not carry `parser`. A function in `ast_attribute.go` rarely carries `astAttribute`. Then propose to fix the file's namespace, such as `//declscope:namespace parse` on `parser.go`:

1. Ask the owner. Say that every baselined finding of the file comes back, and that you will fix them. The check fails without the answer, so wait
2. When the owner agrees, treat that file as a refactoring the user asked for. [Refactoring a part of the package](#refactoring-a-part-of-the-package) applies to it, `-merge` included. That run also drops the package's other entries that match nothing, which is fine. The diff still only removes lines
3. When the owner declines, go to the declined row of the table above

**A vocabulary entry is the exception, for a Go idiom.** Some names follow a convention that never spells the file's name. In `options.go`, the functional options `WithTimeout` and `WithRetry` carry `With`, not `options`. List that word, as in `options: [with]`. It changes no baseline keys. It applies to every package under that config, so say so when you propose it. Do not use a vocabulary entry for a name that could carry a fixed namespace, such as `parse` for `parser`.

**In a feature change, do not rename files only to satisfy a report.** Dropping the prefix from a new file name does make the report go away. It also leaves the package with two ways of naming its files. Renaming the package's files is a reorganization. Do it when the user asks, as the next section explains.

### Refactoring a part of the package

Inside the scope the user named, the package's layout no longer binds you. The goal of the refactoring decides what you change. The findings your changes bring back are yours to fix. Clearing the findings you did not touch is a separate request, even for a whole package. Apply the rest of this skill to what you change:

- Put each declaration in a file named for its concept, and move it there when it is not
- Rename a file whose name does not match its contents, with its test files
- Set `//declscope:namespace` on an existing file whose name reads well but makes poor names, such as `parser.go`. Give its test files the same directive. The request is the owner's permission for the baseline keys it changes
- Widen only what is shared on purpose

**Keep to the scope.** A file outside it keeps its name and its baselined findings, even when the same fix would apply there. Mention those files as candidates for a later change. Update their uses of what you renamed, and nothing else in them. A new file you create for a declaration in the scope is in the scope.

**A file that receives moved code is in scope for that code only.** Keep its name, its namespace and its other declarations. Rename the moved code to carry that file's namespace. When no natural name does, leave the code where it was. Changing the receiving file's namespace would widen the scope.

For example, the user asks to clean up `parser.go` in a package that names its other files `ast_*.go`. None of them has a namespace directive, so `ast_attribute.go` has the namespace `astAttribute`:

| Change | Do it? |
| --- | --- |
| Set `//declscope:namespace parse` on `parser.go` and `parser_test.go`, so `parseExpression` carries it | Yes, when its names read badly with `parser`. Every baselined finding of `parser.go` then comes back, and you fix them |
| Fix the findings that come back for `parser.go`'s declarations | Yes. They are inside the scope |
| Move `parseAttribute` into `ast_attribute.go`, beside the type it builds | Only when a natural name carries `astAttribute`, which is rare. Otherwise it stays in `parser.go` |
| Set `//declscope:namespace attribute` on `ast_attribute.go`, so that the moved function fits | No. That changes the receiving file's namespace |
| Rename `ast_body.go` to `body.go` | No. It is outside the scope. Mention it as a later change |

**Fix the findings your refactoring brings back.** Moving a declaration, renaming it, or renaming its file changes its baseline keys, so its baselined findings report again. They are inside the scope, so fix them as you would a new report. Do not record them in the baseline again.

Then drop the entries your refactoring settled. Run this only after `declscope ./...` and `declscope shrink ./...` both report nothing, since the baseline records `shrink`'s findings too:

```console
declscope baseline -merge ./path/to/pkg
git diff -- '*baseline*'
```

`-merge` keeps the other packages' entries. The command names each file it wrote, which may differ from `.declscope-baseline.yaml` when a config names its own. The diff should remove lines and add none. An added line is a finding recorded instead of fixed. Show the diff to the user with the change.

## Common changes

| Change | What declscope does | Do this |
| --- | --- | --- |
| Add a method to a type, in another file | The method takes its own file's namespace. Each private field it reads is reported at the field, in the type's file | Put the method in the type's file. If the two files are one unit, give the method's file the type's namespace with `//declscope:namespace`. Do not widen the fields |
| Add a field that another file reads | Under a `//declscope:shared` type it is shared already. With nothing stated, the use is reported | When the type is unexported and other files spell it, write `//declscope:shared` on the type. See [Shared structs: private fields](#shared-structs-private-fields). Otherwise, exported types included, write `//declscope:shared // <who reads it>` on each field that is read, however many there are |
| Move code between files | Its namespace changes, and so does the name it must carry | Rename it, move its tests with it, and reread each `// reason` that names a file |
| Split a file | Each new file is a new namespace, so calls between the halves are reported | Split along a real boundary. If the halves are one unit, give them one `//declscope:namespace` |
| Export a name to get past a report | An exported name takes shared scope, so the report goes away | Do not. In `internal/`, `declscope shrink` reports it. Elsewhere it becomes public API |
| Add a helper used only by generated code | Generated files are not read. The `surplus` rule stands down in a package holding one, so nothing is reported | Nothing to write. Never put directives in generated files, since they are not read. An ignore would silence nothing |

In a package whose findings are baselined, a `//declscope:namespace` on an existing file changes its baseline keys. Outside a refactoring the user asked for, propose it instead. Moving or splitting existing code there is a refactoring too. See [Changing a package that has a baseline](#changing-a-package-that-has-a-baseline).

## Naming

This applies when the naming rule is on. The namespace may sit anywhere in the name, but it must start a word. Its right edge may fall inside a word: `loadUserID` carries `user`, and `poweruserID` does not.

Fields, interface methods, and methods written in their type's file are exempt. A method in another file must carry that file's namespace. So do helpers in a test file. `TestXxx`, `BenchmarkXxx`, `FuzzXxx` and `ExampleXxx` in a `_test.go` file are exempt, since `go test` finds them by name.

**Prefer natural word order.** With a verb namespace, a prefix reads as a command: `collectAddFunc` is an instruction, and `addFuncToCollection` is a name. An entry point is the exception, since `collectFiles` already says what it does.

**A verb prefix can promise a side effect the code does not have.** A name that starts with a verb reads as that action. In `delete.go`, a function that only checks whether a user may be deleted must not be called `deleteUser`. A reader expects it to delete the user. Name it for what it does, and fit the namespace in elsewhere: `canDeleteUser`, `usersToDelete`. Keep the verb first only for code that performs the action.

A verb file name is still fine. Its regular noun and past forms often carry it already, since the namespace's right edge may fall inside a word. In `collect.go`, `collectedFiles`, `collectionKeys` and `addToCollection` all carry `collect`. A form that changes the verb's spelling, such as `deletion` for `delete`, needs another fit or a vocabulary entry. The two generated forms are the exception. See [When the name spells the namespace in another form](#when-the-name-spells-the-namespace-in-another-form).

### Fitting the namespace in

A prefix is one fit, and often the right one: `userProfile` is a prefix. Do not use it by reflex, though. Choose the place where the namespace reads as part of the phrase:

| Fit it in as | Namespace | Examples |
| --- | --- | --- |
| A modifier before a noun | `user` | `userProfile`, `userRow` |
| The object after a verb | `user` | `loadUser`, `deleteUserByID` |
| A phrase after the main word | `collect` | `addFuncToCollection` |
| An inflected form | `collect`, `store` | `collectedFiles`, `storingKeys`, `storedKeys` |
| A word it begins, ending inside it | `parse` | `SpecifierParser` |

**Then read the name aloud.** A command (`collectAddFunc`), a stack of nouns with no grammar, or a word used against its meaning means another fit is needed. When no fit reads naturally, the name is not the problem. Suspect the file instead: see [An ill-fitting prefix means the declaration is in the wrong file](#an-ill-fitting-prefix-means-the-declaration-is-in-the-wrong-file).

### Carrying the namespace does not make a name accurate

The rule checks that the namespace is in the name. It cannot check that the rest of the name is true. Name the declaration for exactly what it is, then make sure the namespace is in it. Before naming a collection, read the code that fills it.

| Written | Wrong because | Better |
| --- | --- | --- |
| `methodNames`, filled with every function | Plain functions are in it too, not only methods | `funcNames`, or fill it with methods only |
| `collectInfo` | The namespace is the only real word, so the name says nothing | Name what it holds: `collectedFiles` |
| `orderSortKeys` in `order.go`, read by every file | The prefix claims one owner | Move it. See the next section |

When the accurate name reads badly with the namespace in it, do not bend the words. Read it as a signal about placement.

### An ill-fitting prefix means the declaration is in the wrong file

Sometimes the namespace a name must carry does not describe the thing. Then the declaration sits in the wrong file, and renaming it hides that.

For example, a cache that two parts of a package read was placed in `surplus.go`. So it had to be called `surplusLinknamed`, and the other part's call read `c.surplusLinknamed(pass)`, as if it asked the surplus code. It belonged in the package's shared-model file, which is core and shared as a whole. There it is just `linknamed`.

| The prefix you must add | Do this |
| --- | --- |
| Describes what the thing is | Keep it |
| Belongs to one caller, but several parts use the thing | Move it to a file named for its concept, or to the shared-model file. A concept file often carries the name for free: `linknamed` carries `linkname` |
| Reads badly for most names in the file | Often the file name is wrong. See [What each shape means](../declscope-adoption/SKILL.md#what-each-shape-means) |

This reasoning holds in a package laid out for declscope. In a feature change to a package whose findings are baselined, the file name may follow the package's own convention instead. On a new file, keep that name and set the namespace. On an existing file, propose a namespace to the owner. [Adding a feature](#adding-a-feature) explains both.

### Name a test file for the code it calls

A test file's namespace comes from its own file name: `collect_test.go` is `collect`. So name a test file for the source file whose unexported code it calls, not for the behavior it tests. A test of ordering that calls `collect.go` goes in `collect_test.go`, not `order_test.go`.

The namespace does not follow the subject's `//declscope:namespace`. If `user_extra.go` carries `//declscope:namespace user`, write the same directive on `user_extra_test.go`. A test of a core file joins the core by itself: `client_test.go` joins a core `client.go`. Nothing else joins, so the core takes no GOOS sibling such as `client_linux.go`. A test file with its own `//declscope:namespace` keeps that namespace instead.

| The test calls | Put it |
| --- | --- |
| Unexported code of one file | That file's `_test.go` |
| Unexported code of two files | Split the test, or test through a shared entry point |
| A helper shared by several test files | A test file named for the helper, such as `fixture_test.go`. Put `//declscope:shared // <which tests use it>` on the helper |
| Only exported names | Anywhere, including `package x_test` |

**Uses only in `_test.go` files mean the test is misplaced.** Move the test. Never widen production code for a test, and never let `-fix` insert `//declscope:shared` for one.

### When the name spells the namespace in another form

Only the namespace's own spelling and two generated forms carry it: a final `e` dropped before `ing` (`store` → `storing`) and `y` turned to `i` (`apply` → `applies`, `applied`). Generation only makes the namespace longer. Nothing else is guessed:

| File | Name | Carried | Why |
| --- | --- | --- | --- |
| `store.go` | `storingKeys` | Yes | A generated form |
| `tracing.go` | `traceValue`, `tracerType` | No | The stem of an inflected file name is not derived |
| `walking.go` | `walkSpeed` | No | Same |
| `index.go` | `indicesSorted` | No | An irregular form |

**Prefer the base form of a word for a new file name.** It is a recommendation, not a requirement. `trace.go` is carried by `traceValue`, `tracerType` and `tracingStart`. `tracing.go` is carried only by the last. Generated forms and the free right edge both run from the namespace to longer words, so the base form reaches every form the inflected file name reaches, and more. The same holds for `entry.go` over `entries.go`, `walk.go` over `walking.go`, and `user.go` over `users.go`. An inflected file name is allowed. It only fits declscope worse, so choose it when it reads clearly better.

For an existing file, renaming it to the base form keeps every name that carried the old namespace. It still changes the namespace, so it is not free. Rename its test files too, and update any `//declscope:namespace` that names it and each `// reason` that names the file. Its baselined findings report again. Fix them, as [Refactoring a part of the package](#refactoring-a-part-of-the-package) explains. Do not rename an existing file only for this. Mention the rename as an option, unless the file is in a refactoring the user asked for. When it is too wide, or the owner prefers the current name, a vocabulary entry is the smaller change.

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

## Directives

Only the exact form `//declscope:name` is a directive: a line comment, no space after the slashes or the colon. Anything else starting with `declscope:` is reported as malformed.

| Directive | Where it goes | Note |
| --- | --- | --- |
| `//declscope:shared`, `//declscope:private` | The doc comment of the declaration, or trailing its first or last line | On a block, it reaches every spec in it |
| The same, file-level | Above the package clause | Reaches every declaration in the file, members included |
| `//declscope:ignore <rules>` | The same places as either of the above. Rules are comma-separated | A bare ignore silences every rule except `overexported` |
| `//declscope:namespace <name>` | Above the package clause | Below it, the `directive` rule reports it, and it has no effect. It also keeps a file name that follows the package's convention. On an existing file, it changes the baseline keys |
| `//declscope:core` | Above the package clause | Sets the namespace only. It shares nothing |

**Write the reason after `//`.** `//declscope:shared // rename.go reads it` and `//declscope:ignore boundary // why` are directives with reasons. Any other text after the name is read as an argument. `//declscope:ignore boundary because ...` is reported as an unknown rule.

**A directive records a decision.** Write one only when it changes something, and say why. Under the default `unused: strict`, a directive that restates the scope already in force is reported.

**Share a whole file only when the file exists to be shared.** A directive on a file or a block reaches everything inside it. Each exception then needs a directive of its own. A file-level `//declscope:shared` also makes every new declaration in the file start shared. So it should describe what the file is, not who happens to use it today.

| The file or block | Write |
| --- | --- |
| Is shared by nature, such as a utility file or the package's shared model, and a few declarations are its internals | `//declscope:shared` on the whole, and `//declscope:private` on each of the few internals |
| Happens to have many declarations used from other files right now | `//declscope:shared` on each of them, and nothing on the whole |
| Has only a few declarations used from other files | `//declscope:shared` on each of those few, and nothing on the whole |

Do not share a whole file to share a few declarations, then mark the rest `//declscope:private`. The file then states the opposite of what it is. A type is decided differently. An unexported type gets the directive when other files spell it, not by how many of its fields cross. An exported type is shared already, so it gets none. See [Shared structs: private fields](#shared-structs-private-fields).

## Shared structs: private fields

When another namespace uses a struct, give each field the smallest scope it needs.

| Declaration | Directive |
| --- | --- |
| An unexported struct type, spelled from another namespace | `//declscope:shared`. Its fields inherit it |
| An exported struct type | None. It is shared already. Its unexported fields stay private, unless a file-level directive says otherwise |
| A field no other namespace reads, under a `//declscope:shared` type | `//declscope:private`, after its doc comment and a `//` line |
| A field another namespace reads, under a `//declscope:shared` type | None |
| A field another namespace reads, under any other type | `//declscope:shared // <who reads it>` |
| An embedded field | None. It is not a target, and a directive there is reported as unused |

**Do not restate the type's scope on a field.** It binds nothing and is reported as unused.

**Put the private fields last,** as [Order by reach](#order-by-reach) says. The fields other files read are the struct's interface, so they come first:

```go
// callee is a resolved call target.
//
//declscope:shared
type callee struct {
	// obj is the declared function or method, when there is one.
	obj *types.Func
	// inputs are the values passed.
	inputs []ssa.Value
	// fn is the function called, when it is known statically.
	//
	//declscope:private
	fn *ssa.Function
}
```

Under a `//declscope:shared` type with `surplus: strict`, declscope reports each field no other namespace reads. Where `surplus` is off or stands down, find them by hand. Mark every field `//declscope:private`. Then delete the directive from each field reported as used from another namespace.

**A type nobody else spells needs no directive,** even when other files read its fields. Callers that only get it from a constructor do not spell it. Keep the type private. Write `//declscope:shared // <who reads it>` on each field that another file reads, and nothing on the rest.

When no other file reads its fields either, `//declscope:shared` on the type is reported:

```text
//declscope:shared on hidden, hidden.a: no use from another namespace is visible to declscope
```

## Order by reach

**Order declarations from the widest reach to the narrowest.** A reader meets the file's interface first: what other packages use, then what other files use, then how it works. declscope does not check this order. It is a convention for code you write.

| Reach | Declarations |
| --- | --- |
| 1. Other packages | Exported, and not `private` |
| 2. Other files of the package | Unexported, and `shared` by any directive in force: its own, its block's, its type's or the file's |
| 3. Its own namespace | `private`, exported or not |

Scope decides the reach, not the case of the first letter. An exported declaration marked `//declscope:private` goes with the private ones, since its author said no other file uses it.

**Order only inside a sequence the file already has.** Keep the order of kinds. Within each reach, keep the existing order, so declarations that sit together stay together.

**A type and the constructors and methods right after it are one unit.** Move the unit as a whole, at the type's reach. Do not move the type out alone, and do not move another declaration into the unit. Inside it, the type comes first, then its constructors, then its methods, each ordered by reach.

| Sequence | Order by reach |
| --- | --- |
| The fields of a struct, the methods of an interface | Yes |
| A run of adjacent declarations of one kind, such as consecutive functions or one `const` block | Yes |
| A run of type units | Yes, each unit as a whole |
| The constructors of a unit, and separately its methods | Yes, inside the unit |
| Declarations of different kinds | No. Do not move a declaration past one of another kind |
| Sections marked by a comment, such as `// ===` or `// Param Types` | No. Order inside each section |

**A private helper may stay right after its users.** When the declarations that use it sit next to each other, put it directly after the last of them, or after another helper already placed there. Reading from the top then follows the calls. A helper whose users are spread through the file goes in reach order.

```go
func Resolve(dir, explicit string) (Options, error) { ... }

func ResolveForBaseline(dir, explicit string) (Options, string, error) { ... }

// resolve is used only by the two above, so it stays after them.
func resolve(dir, explicit string) (Options, string, error) { ... }

func Find(dir string) string { ... }
```

Fields have no such exception. A private field goes after the shared ones, even the field the struct is built around.

Do not reorder where the order is observable:

| Order is observable through | Effect |
| --- | --- |
| `iota`, or a `const` spec with its expression omitted | Each value follows its position |
| Unkeyed composite literals | `callee{f, in, fn}` binds fields by position |
| Positional or binary encodings | The wire format follows the field order |
| `unsafe` offsets | `unsafe.Offsetof` changes |
| 64-bit atomics | They rely on first-word alignment on 32-bit platforms |
| Package-level `var` initializers with side effects and no dependencies between them | They run in declaration order |
| Several `init` functions in one file | They run in source order |

**Keep an order the reader relies on.** Some orders change nothing for the compiler but still say something. Keep these as you would an observable order:

- Steps listed in the order they run.
- Values that follow an outside order, such as a page's layout or a protocol's fields.
- A comment that introduces the declarations after it. Move it with them, or leave the group in place.

When you are unsure, leave the sequence and say so in your report.

Where the repository's linters fix an order, such as `decorder` or `funcorder`, follow them first. `funcorder`'s exported-before-unexported methods is the same direction, so order by reach inside it.

Test files and generated files are out of scope. `TestXxx` functions are exported but called by nobody.

**When your change moves a declaration to another reach, move the declaration too.** That is the default, for a new `//declscope:shared`, a `//declscope:private` from `surplus`, or a name `shrink` unexported. Say in your report that you moved it: the move makes the diff larger, and the owner may prefer it in place. Do not reorder declarations your change does not touch. In a feature change to a package whose findings are baselined, leave it in place and mention the move.

## Acting on a diagnostic your change caused

**A boundary report points at the declaration you reached, not at your code.** Your use is listed under it as `used here, in namespace ...`. A directive or an ignore that answers it goes on the declaration.

| Report | First question | Usual answer |
| --- | --- | --- |
| `X is private to namespace "a"` (or `to the core namespace`), `but is used from namespace "b"`, or `X is declared private by ...` | Should your code live beside `X`? | Move your code beside `X`. If `X` is shared on purpose and sits in one caller's file, move it to a file named for its concept first. Then write `//declscope:shared // why` on `X`. When `X` states its own `//declscope:private`, that was a decision: move your code, and do not flip the directive. In a feature change to a baselined package, see [Adding a feature](#adding-a-feature) first |
| `//declscope:shared on X: no use from another namespace is visible` | Did the use that justified it go away? | Delete the directive |
| `X takes shared scope from ..., but no use ...` | Does any other file read `X`? | Accept `-fix`'s `//declscope:private` on `X`. Move `X` after the declarations of wider reach, except in a feature change to a package whose findings are baselined. See [Order by reach](#order-by-reach). When the directive is a file's or a block's, and most of its declarations get this report, it should not share the whole. Delete it, and write `//declscope:shared` on the few that other files use |
| `unused //declscope:...` | Does the directive still change anything? | Delete it |
| `X does not carry namespace "a"` | Is the name accurate, and is `X` in the right file? | See [Naming](#naming). Rename only when both answers are yes |
| `X is exported, but nothing outside ... uses it` (`declscope shrink`) | Does another package really need it? | See [Reading what shrink reports](#reading-what-shrink-reports) |

**`-fix` widens. It does not draw boundaries.** On a boundary report, `-fix` inserts `//declscope:shared` instead of moving code. That also leaves the declaration in its first caller's file, with that caller's prefix. Read `-fix -diff` before `-fix`. Its rename is offered only when it is proven safe, but it cannot tell whether the name was accurate.

### Reading what shrink reports

`declscope shrink` reports exported declarations of `internal/` packages that nothing outside their package uses. It is a subcommand, not a rule the analyzer runs, so a clean `declscope ./...` says nothing about it.

| Report | What to do |
| --- | --- |
| `... uses it` and nothing more | The fix is offered. Apply it with `declscope shrink -fix <that package>` |
| `... (no fix: <reason>)` | A use may exist that `shrink` cannot prove, or the rename is unsafe. **Do not unexport it by hand.** Read the reason first |
| `... only the external tests of <pkg> use it` | Keep it exported. Add `//declscope:ignore overexported // <why>` when the tests use it on purpose |
| `... (no fix: a build-excluded file of another package may use it)`, where that file uses it under another build tag | Keep it exported. Add `//declscope:ignore overexported // <file, behind which tag>`. One ignore passes every configuration |
| `declscope shrink: not judged: <pkg>: <reason>` on stderr | That package was not checked. It is not clean |
| `declscope shrink: warning: "<pattern>" matched no packages` on stderr | The pattern checked nothing. Fix the pattern, as you would for `go vet` |
| `declscope shrink: not judged: <n> package(s) outside the main module` on stderr | The patterns named packages `shrink` never judges, such as `std`. Nothing to do |

**A package not judged is not a package with nothing to report.** `shrink` stands down wherever an importer could be unseen:

- outside `internal/`, and in `package main`;
- beside assembly or cgo;
- under an `internal/` whose tree a nested module may import into, when that module fails to load or reads this one from elsewhere, such as a published version;
- under an `internal/` whose tree a module of the build list may import into, when the build reads that module from elsewhere, such as a published version.

A copy from elsewhere that a nested module builds keeps the reports in its range, but withholds their fixes, and the reason names the copy.

The exit status ignores those packages.

**The values of an enum are judged as one set.** A `const` block of 2 or more constants, all of one exported type of the package, is a set. While the type is not reported, or another package uses any of its values, no value is reported. Otherwise every value is reported with the type, and is fixed only where all of them are. A constant of the type outside such a block, such as a lone default, is judged on its own. To keep it with the set, put it in the block. A report on a block that mixes in another constant says to split the block. Do not add an ignore per value.

A bare `//declscope:ignore` does not reach `overexported`. Name the rule. Names written as strings, such as a template field or `reflect.Value.MethodByName`, are outside what `shrink` can see. When no interface carries the value there, add the ignore with the reason.

## Do not hide a report

| Tempting | Why not |
| --- | --- |
| `rules.boundary: off` | Turns off the rule the tool exists for |
| `//declscope:core`, or a file-level `//declscope:shared` on a file that is not shared by nature | Hides the boundaries instead of stating them |
| Exporting a name | Moves the problem to `shrink`, or into the public API |
| A name bent to carry the namespace | Passes the check and misleads the reader |
| Regenerating the baseline over new findings | Records your new crossing as if it were old |
| `//declscope:ignore` without a reason | The next reader cannot tell a decision from a shortcut |

**Config changes are the owner's decision.** Ask before writing or changing one.

## Traps

**A package that does not compile is skipped, not checked.** declscope prints `analysis skipped due to errors in package` and exits 1, not 3. That package adds no diagnostics, so its count reads as zero. Run `go vet ./...` first, which also compiles the tests declscope reads.

**A bulk rename reaches further than intended.** A `\bname\b` substitution across every `.go` file will hit `keys`, `named` and `check` in testdata and unrelated packages. Limit the paths, then read `git status`.

## Before you finish

```bash
go vet ./... && go test ./...
declscope shrink ./...   # when the repository has internal/ packages
declscope ./...
```

Use the config the repository's CI uses. Some repositories check themselves with an explicit `-config`.
