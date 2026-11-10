---
name: declscope-authoring
description: "Write or change Go code in a repository that runs declscope, which shows as a .declscope.y*ml or baseline file, //declscope: comments, or declscope in CI. Read this before adding, naming or moving a declaration, a helper that several files use, or a test. Read it too before splitting a file, writing a //declscope: directive, or acting on a declscope diagnostic. Covers where code belongs, naming it accurately, and fixes that hide a problem."
license: MIT
x-embedded-by: declscope
x-embedded-version: 0.25.0
x-embedded-at: "2026-11-10T14:50:10Z"
x-embedded-digest: "sha256:b0380c5d8da51087605f258f66a87fcc3131406996d7c95457996ca607f83831"
---

# Writing code under declscope

Written against **declscope 0.25.0**. Check the version first with `declscope -V=full`: this describes how that release behaves, not how an older one does.

This skill is for everyday work in a repository that already runs declscope. Introducing it, choosing its configuration, and clearing the whole repository's baseline are in [declscope-adoption](../declscope-adoption/SKILL.md), installed beside this one. Changing one package whose findings are baselined, including clearing its entries, is in [Changing a package that has a baseline](#changing-a-package-that-has-a-baseline). What each directive and config key means is in [the README](https://github.com/mpyw/declscope#readme).

**Read each rule with its reason.** Most rules here are defaults for a situation, and say why. When your situation differs from the one a rule was written for, follow the reason, not the wording. For example, keeping existing code in place protects a feature change from a half-done reorganization. When you can finish the reorganization, that reason is gone. The exceptions are in [Do not hide a report](#do-not-hide-a-report): they hold everywhere.

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

**The naming rule asks a name to carry its file's namespace**: `loadUser` in `user.go` carries `user`. Advice in this skill that depends on the rule says "under the naming rule". When the rule is on, read [naming.md](naming.md) too, before you name a declaration or write `//declscope:namespace`. It covers how a name carries its namespace, what to do when it cannot, and vocabulary entries.

## Where a new declaration goes

**Put it in the file whose concern it is.** That choice decides its namespace, so it decides who may use it. Under the naming rule, it also decides what the name must carry. The directive and the name follow from it.

| The declaration is used by | Put it | Scope |
| --- | --- | --- |
| Code in one file | That file | Default. No directive |
| Several files, as a shared helper on purpose | A file named for its concept, not the file of its first caller | `//declscope:shared // <who uses it and why>`. A file-level one only when the file is shared by nature. See [Directives](#directives) |
| Most files of the package, as its shared model | A core file: `//declscope:core` plus a file-level `//declscope:shared` | Package-wide, from the file directive |
| Another package | Wherever it belongs, exported | Default for exported names |

**A core file does not share its declarations by itself.** `//declscope:core` sets only the namespace, and exempts the file from the naming rule. Its unexported declarations are still private to the core. A core file whose declarations are shared also needs a file-level `//declscope:shared`.

**Before widening, ask whether the code is in the wrong file.** A boundary report on your new use says a file reached into another namespace. Often the function you are writing belongs beside what it uses. When the declaration is shared on purpose, check that it sits in a file named for its concept before you write the directive. When you add a feature to a package whose findings are baselined, keep to its layout unless you can finish the restructuring. See [Changing a package that has a baseline](#changing-a-package-that-has-a-baseline).

## Changing a package that has a baseline

A baseline is a file, usually `.declscope-baseline.yaml`, that records the findings a repository had when it adopted declscope. declscope does not report what it records. Only new findings fail the check: `declscope` and `declscope shrink`, as CI runs them.

This section is for a package whose findings are recorded in a baseline. Such a package was laid out before declscope. A baseline file usually covers a whole repository, so check the package itself: `declscope survey ./path/to/pkg` shows a `baselined` count above zero.

What you do depends on what you were asked:

| The request | Do this |
| --- | --- |
| Add a feature or fix a bug | Keep the package's layout, or restructure what you touch when you can finish it, as [Adding a feature](#adding-a-feature) explains |
| Refactor some files, a part of the package, or the whole package | Inside that scope, follow this skill as in a package laid out for declscope. See [Refactoring a part of the package](#refactoring-a-part-of-the-package) |
| Clear one package's baseline | A refactoring of the whole package that fixes all its findings. See [Clearing one package's baseline](#clearing-one-packages-baseline) |
| Introduce declscope, or clear the whole repository's baseline | Follow [declscope-adoption](../declscope-adoption/SKILL.md) |

A request to refactor widens how far you may restructure. Read it from what the user wants changed, however it is worded: "clean up `parser.go` while you are there", "refactor this package", or "this function is hard to read". "Fix the lint errors" asks for those fixes, moves included, and no more. When the scope is unclear, such as "tidy this up", ask which files it covers.

The owner is the user you are working for. When a step needs the owner's answer, ask and wait. Never finish with a failing check, a bent name or an unexplained ignore. When nobody can answer, as in a non-interactive run, [When nobody can answer](#when-nobody-can-answer) says what to do.

### Adding a feature

A bug fix follows the same rules. **Add new code the way the package already does it.** Some packages keep a layer in one file, such as every parse function in `parser.go`. Others name files with a prefix, such as `ast_object_cons_expr.go`.

**Restructure only as far as you can finish.** You may move or rename the code you touch when the result is whole. Every finding that comes back is fixed. The package does not end up mixing two conventions. The diff stays small enough to review with the feature. Renaming one `ast_*.go` file without its prefix, or moving some of `parser.go`'s parse functions but not the rest, mixes two. A half-done reorganization is worse than none. When the change would not be whole, keep the layout, and propose the reorganization instead.

| The package | Do this |
| --- | --- |
| Names its files with a shared prefix, and you add a new concept | Name the new file the same way. Under the naming rule, set its namespace with `//declscope:namespace` so the names carry it without the prefix |
| Keeps one layer in one file | Put the new functions in that file. A method goes in its type's file, as [Common changes](#common-changes) says |
| Already has a file for the concept you add | Put the new code there |

Here is a new file for the first row, under the naming rule. `parser.go` calls `newFunctionCallExpr`. Its test file, `ast_function_call_expr_test.go`, takes the same directive.

```go
// ast_function_call_expr.go
//declscope:namespace functionCallExpr
package hclwrite

// FunctionCallExpr carries functionCallExpr, and the file keeps its name.
type FunctionCallExpr struct{ /* ... */ }

//declscope:shared // parser.go builds these
func newFunctionCallExpr( /* ... */ ) *FunctionCallExpr { /* ... */ }
```

**A baseline key is the declaration's namespace and name.** Moving a declaration, renaming it or its file, or setting a namespace on the file changes the key. Its recorded findings come back. Fixing them is part of the move. Move or rename an existing declaration only when you can fix what comes back, as above. On a file you create, there is no such cost.

When the new code uses a private declaration of another file, declscope reports it on that declaration. First ask whether the new code belongs beside it. New code costs nothing to place, so put it in that file when it does. Otherwise:

| The declaration | Do this |
| --- | --- |
| Has no directive | Ask whose concept it is. When it belongs to its file's concept, such as `parseExpr` in `parser.go`, write `//declscope:shared // <who uses it>` on it and leave it. When its file was only its first caller, such as a general helper now used twice, move it to a file named for its concept. That file may exist already, or be new. Then share it. Moving it changes its baseline key, so do it only when you can finish, as above |
| Has a bare `//declscope:private` of its own, such as `-fix` writes for `surplus` | It only records that nothing else read it yet. Delete it, and treat the declaration as in the row above |
| Is private through a directive on its file, block or type | That decides the whole, not this declaration. Write `//declscope:shared // <who uses it>` on it as a stated exception, as the first row does |
| Has `//declscope:private` with a reason | That is a decision. Ask the owner whether it still holds. When it no longer does, replace it with `//declscope:shared // <who uses it>`. When it does, or nobody can answer, put the new code beside it after all. When the use sits inside an existing function, an ignore with its reason is the last resort |

When the naming rule reports that a new name cannot carry the file's namespace, see [In a package that has a baseline](naming.md#in-a-package-that-has-a-baseline).

When your change happens to remove a recorded finding, its entry now matches nothing and does no harm. Drop it as [Updating the baseline](#updating-the-baseline) says, or leave it and mention it in your report.

### Refactoring a part of the package

Inside the scope the user named, the package's layout no longer binds you. The goal of the refactoring decides what you change. Apply the rest of this skill to what you change:

- Put each declaration in a file named for its concept, and move it there when it is not
- Rename a file whose name does not match its contents, with its test files
- When a file's name fits but its namespace makes the names awkward under the naming rule, set a better one with `//declscope:namespace`. Do the same on its test files
- Widen only what is shared on purpose

**The findings that come back are the work.** Every move and rename above changes keys, so recorded findings report again. Fix them as you would a new report, and do not record them again. Keeping a declaration in place, or marking it `//declscope:shared` instead of moving it, keeps the layout the user asked you to change.

**Keep to the scope.** A file outside it keeps its name, its namespace and its recorded findings. Update its uses of what you renamed, and mention it as a candidate for a later change. Under the naming rule, code you move into such a file must carry its namespace. When no natural name does, leave the code where it was, or give it a new file. A new file is in the scope, so choose its name and namespace to fit what it receives.

For example, the user asks to clean up `parser.go` in a package that names its other files `ast_*.go`. None has a namespace directive, so `ast_attribute.go` has the namespace `astAttribute`:

| Change | Do it? |
| --- | --- |
| Set `//declscope:namespace parse` on `parser.go` and `parser_test.go`, so `parseExpression` carries it | Yes, when its names read badly with `parser`. Fix the findings that come back |
| Move `parseAttribute` into `ast_attribute.go`, beside the type it builds | Yes without the naming rule. Under it, only when a natural name carries `astAttribute`, which is rare |
| Set `//declscope:namespace attribute` on `ast_attribute.go`, so that the moved function fits | No. That file is outside the scope |
| Rename `ast_body.go` to `body.go` | No. Mention it as a later change |

#### When the request keeps the API

Exported names cannot be renamed then. The same holds outside `internal/` unless the request says the API may change, since importers you cannot see use those names. Where the naming rule reaches them, plan moves around their names:

| Change | Do this |
| --- | --- |
| Move exported declarations to a new file | Choose its namespace so they carry it as they are. `RequiredParam` and `OptionalParam` move from `server.go` to `params.go` with `//declscope:namespace param` |
| Rename a file, or set a namespace on it | First check that its exported names carry the new namespace. When they cannot, leave that file |
| `shrink` reports an existing exported name | Keep it with `//declscope:ignore overexported // <why it stays exported>`, and mention it |

#### Clearing one package's baseline

Recorded findings are not reported, not even by `shrink`. To see them, move aside the baseline file that records the package, and fix the package until `declscope` and `declscope shrink` report nothing for it. The request allows changing exported names under `internal/`, unless it keeps the API. Then put the file back, and update it as below.

#### Updating the baseline

After a refactoring, drop the entries it settled. Run this only when `declscope` reports nothing for the package, and `declscope shrink` too where the repository runs it:

```console
declscope baseline -merge ./path/to/pkg   # add -shrink=false where shrink does not run
git diff -- '*baseline*'
```

The diff should remove lines and add none. An added line is a finding recorded instead of fixed. Show the diff to the user with the change. When the baseline ends up empty, delete it.

### When nobody can answer

In a non-interactive run, do not stop. Take the step this table gives, and say in your report what you did and why:

| The step that needs an answer | Do this |
| --- | --- |
| The scope of a refactoring is unclear | Take the files the request names, or else the files your change touches |
| A vocabulary entry | Add it |
| Any other config change | Do not make it |
| Anything else | Carry out the option you would recommend. When nothing else passes without a bent name, an ignore with its reason is the last resort |

## Common changes

| Change | What declscope does | Do this |
| --- | --- | --- |
| Add a method to a type, in another file | The method takes its own file's namespace. Each private field it reads is reported at the field, in the type's file | Put the method in the type's file. If the two files are one unit, give the method's file the type's namespace with `//declscope:namespace`. Do not widen the fields |
| Add a field that another file reads | Under a `//declscope:shared` type it is shared already. With nothing stated, the use is reported | When the type is unexported and other files spell it, write `//declscope:shared` on the type. See [Shared structs: private fields](#shared-structs-private-fields). Otherwise, exported types included, write `//declscope:shared // <who reads it>` on each field that is read, however many there are |
| Move code between files | Its namespace changes, and under the naming rule, so does the name it must carry | Move its tests with it, and reread each `// reason` that names a file. Under the naming rule, rename it to carry the new namespace |
| Split a file | Each new file is a new namespace, so calls between the halves are reported | Split along a real boundary. If the halves are one unit, give them one `//declscope:namespace` |
| Export a name to get past a report | An exported name takes shared scope, so the report goes away | Do not. In `internal/`, `declscope shrink` reports it. Elsewhere it becomes public API |
| Add a helper used only by generated code | Generated files are not read. The `surplus` rule stands down in a package holding one, so nothing is reported | Nothing to write. Never put directives in generated files, since they are not read. An ignore would silence nothing |

In a package whose findings are baselined, a `//declscope:namespace` on an existing file, or moving or splitting existing code, changes baseline keys. Do it only when you can finish what comes back. See [Changing a package that has a baseline](#changing-a-package-that-has-a-baseline).

## Name a test file for the code it calls

A test file's namespace comes from its own file name: `collect_test.go` is `collect`. So name a test file for the source file whose unexported code it calls, not for the behavior it tests. A test of ordering that calls `collect.go` goes in `collect_test.go`, not `order_test.go`.

The namespace does not follow the subject's `//declscope:namespace`. If `user_extra.go` carries `//declscope:namespace user`, write the same directive on `user_extra_test.go`. A test of a core file joins the core by itself: `client_test.go` joins a core `client.go`. Nothing else joins, so the core takes no GOOS sibling such as `client_linux.go`. A test file with its own `//declscope:namespace` keeps that namespace instead.

| The test calls | Put it |
| --- | --- |
| Unexported code of one file | That file's `_test.go` |
| Unexported code of two files | Split the test, or test through a shared entry point |
| A helper shared by several test files | A test file named for the helper, such as `fixture_test.go`. Put `//declscope:shared // <which tests use it>` on the helper |
| Only exported names | Anywhere, including `package x_test` |

**Uses only in `_test.go` files mean the test is misplaced.** Move the test. Never widen production code for a test, and never let `-fix` insert `//declscope:shared` for one.

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
| Is shared by nature, such as a utility file or the package's shared model, and a few declarations are its internals | `//declscope:shared` on the whole, and `//declscope:private // <why it stays inside>` on each of the few internals |
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

**When your change moves a declaration to another reach, move the declaration too.** That is the default, for a new `//declscope:shared`, a `//declscope:private` from `surplus`, or a name `shrink` unexported. Say in your report that you moved it: the move makes the diff larger, and the owner may prefer it in place. Do not reorder declarations your change does not touch. In a file not ordered by reach, such as many laid out before declscope, leave it in place and mention the move.

## Acting on a diagnostic your change caused

**A boundary report points at the declaration you reached, not at your code.** Your use is listed under it as `used here, in namespace ...`. A directive or an ignore that answers it goes on the declaration.

| Report | First question | Usual answer |
| --- | --- | --- |
| `X is private to namespace "a"` (or `to the core namespace`), `but is used from namespace "b"`, or `X is declared private by ...` | Should your code live beside `X`? | Move your code beside `X` when it belongs there. Otherwise act on `X`'s directive as the table in [Adding a feature](#adding-a-feature) says, with or without a baseline. Inside a refactoring the user asked for, you may change a decided one, and say why |
| `//declscope:shared on X: no use from another namespace is visible`, or `the file's //declscope:shared: no use ...` | Did the use that justified it go away? | Delete the directive |
| `X takes shared scope from ..., but no use ...` | Does any other file read `X`? | When the directive is a file's or a block's, and most of its declarations get this report, it should not share the whole. Delete it, and write `//declscope:shared` on the few that other files use. Otherwise accept `-fix`'s `//declscope:private` on `X`, alone: take that edit from `declscope -fix -diff`, since a whole `-fix` run also widens every boundary report. When the file is ordered by reach, move `X` after the declarations of wider reach. See [Order by reach](#order-by-reach) |
| `unused //declscope:...` | Does the directive still change anything? | Delete it |
| `X does not carry namespace "a"` | Is the name accurate, and is `X` in the right file? | See [naming.md](naming.md). Rename only when both answers are yes. The name the message suggests is one option, not the answer. See [When `-fix` renames](#when--fix-renames) |
| `X is exported, but nothing outside ... uses it` (`declscope shrink`) | Does another package really need it? | See [Reading what shrink reports](#reading-what-shrink-reports) |

**`-fix` widens. It does not draw boundaries.** On a boundary report, `-fix` inserts `//declscope:shared` instead of moving code. That also leaves the declaration in its first caller's file, with that caller's prefix. Read `-fix -diff` before `-fix`. It renames only under `rules.naming.fix: true`. See [When `-fix` renames](#when--fix-renames).

### When `-fix` renames

With `rules.naming.fix: true`, `-fix` renames every reported name at once. Do not apply it wholesale. The rename is proven safe, but nothing checks that the new name is accurate or that the file is right.

| Step | Do this |
| --- | --- |
| 1 | Read `declscope -fix -diff` |
| 2 | Judge each rename by [naming.md](naming.md). Is the name accurate? Is the declaration in the right file? |
| 3 | Take only the edits you accept. Name or move the rest yourself |

The name the message suggests is one option, not the answer. Without the key, the report carries no fix, and you choose the name.

### Reading what shrink reports

`declscope shrink` reports exported declarations of `internal/` packages that nothing outside their package uses. It is a subcommand, not a rule the analyzer runs, so a clean `declscope ./...` says nothing about it.

**Each report names a declaration.** Decide per name, then fix:

| Report | What to do |
| --- | --- |
| `... uses it` and nothing more | A name you added: unexport it. An existing name: unexport it when the request allows changing exported names. Otherwise keep it, as [When the request keeps the API](#when-the-request-keeps-the-api) says |
| `... only the external tests of <pkg> use it` | Keep it exported. Add `//declscope:ignore overexported // <why>` when the tests use it on purpose |
| `... (no fix: a build-excluded file of another package may use it)`, where that file uses it under another build tag | Keep it exported. Add `//declscope:ignore overexported // <file, behind which tag>`. One ignore passes every configuration |
| Any other `... (no fix: <reason>)` | A use may exist that `shrink` cannot prove, or the rename is unsafe. Read the reason. Do not unexport an existing name by hand: add `//declscope:ignore overexported // <the reason shown>`, and mention it. For a name you added, choose another unexported name yourself |

`declscope shrink -fix <pkg>` unexports every report of the package at once, except those ignored. So write the ignores for the names you keep first, then run it.

Name the rule in each ignore: a bare `//declscope:ignore` does not reach `overexported`.

**Some packages are not judged.** `shrink` stands down wherever an importer could be unseen. A package not judged is not a package with nothing to report.

| Where | Why |
| --- | --- |
| Outside `internal/`, and in `package main` | Any module may import it, or nothing can |
| Beside assembly or cgo | Uses there are not visible |
| Under an `internal/` that a nested module may import | The nested module fails to load, or reads this module from elsewhere, such as a published version |
| Under an `internal/` that a module of the build list may import | The build reads that module from elsewhere, such as a published version |

When such a copy loads, the reports in its range stay, but their fixes are withheld, and the reason names the copy. The exit status ignores the packages not judged. They show on stderr:

| Message on stderr | What to do |
| --- | --- |
| `declscope shrink: not judged: <pkg>: <reason>` | That package was not checked. It is not clean |
| `declscope shrink: warning: "<pattern>" matched no packages` | The pattern checked nothing. Fix the pattern, as you would for `go vet` |
| `declscope shrink: not judged: <n> package(s) outside the main module` | The patterns named packages `shrink` never judges, such as `std`. Nothing to do |

**The values of an enum are judged as one set.** A set is a `const` block of 2 or more constants of one exported type of the package. No value is reported while the type is not reported, or while another package uses any value. Otherwise every value is reported with the type, and fixed only where all of them are. A constant of the type outside the block, such as a lone default, is judged alone: put it in the block to keep it with the set. A report on a block that mixes in another constant says to split the block. Do not add an ignore per value.

**Names written as strings are outside what `shrink` can see**, such as a template field or `reflect.Value.MethodByName`. When no interface carries the value there, add the ignore with the reason.

## Do not hide a report

| Tempting | Why not |
| --- | --- |
| `rules.boundary: off` | Turns off the rule the tool exists for |
| `//declscope:core`, or a file-level `//declscope:shared` on a file that is not shared by nature | Hides the boundaries instead of stating them |
| Exporting a name | Moves the problem to `shrink`, or into the public API |
| A name bent to carry the namespace | Passes the check and misleads the reader |
| Regenerating the baseline over new findings | Records your new crossing as if it were old |
| `//declscope:ignore` without a reason | The next reader cannot tell a decision from a shortcut |

**Config changes are the owner's decision.** Ask before writing or changing one, and see [When nobody can answer](#when-nobody-can-answer) for a run nobody answers.

## Traps

**A package that does not compile is skipped, not checked.** declscope prints `analysis skipped due to errors in package` and exits 1, not 3. That package adds no diagnostics, so its count reads as zero. Run `go vet ./...` first, which also compiles the tests declscope reads.

**A bulk rename reaches further than intended.** A `\bname\b` substitution across every `.go` file will hit `keys`, `named` and `check` in testdata and unrelated packages. Limit the paths, then read `git status`.

## Before you finish

```bash
go vet ./... && go test ./...
declscope shrink ./...   # when the repository runs it
declscope ./...
```

Use the config the repository's CI uses. Some repositories check themselves with an explicit `-config`.
