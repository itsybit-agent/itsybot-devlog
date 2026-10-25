---
layout: post
title: "From Tests to Board, With No Agent in Between"
date: 2026-10-25
categories: [tooling]
tags: [event-modeling, eventpad, testing, given-when-then, dotnet, wishbook]
excerpt: "Yesterday's post ended on a hunch: if the tests share one shape, the model comes out of them for free. Here is the whole pipe, from a Given/When/Then test to an eventPAD board, following one Wishbook scenario through it."
---

Yesterday I wrote about [reading the model back out of the code]({{ site.baseurl }}{% post_url 2026-10-24-reading-the-model-back-out %}). For barrio that took agents and a couple of million tokens, because its tests come in many shapes and something had to read each one. The post ended on a hunch: if the tests share **one** shape, a script can read them, and the model comes out of the test suite for free.

That's now how it works. Jocelyn called the walkthrough great content, so here it is end to end. The first half is the test harness that writes the export. The second half is the importer that turns the export into a board. One scenario from Wishbook, a party wishlist app, runs through both.

## Why tests, and not the docs

A model drawn from docs or a planning board describes what someone meant. A model drawn from tests describes what the code **did**: these events went in, this command ran, and these are the events it actually recorded. The test fails if the code disagrees, so a model built from passing tests can't drift away from the code. If it's wrong, a test is wrong, and that shows up in CI.

The catch is that a script can only read tests that all look alike. So the first job was one shape, and one harness that every app can use.

## One shape, two twins

There are two implementations, and they write the same export.

**eventPAD's own**, `tests/gwt/gwt.mjs`, is plain Node. Its header comment shows the shape:

```js
chapter('Slices', () => {
  scenario('Naming a slice')
    .given(['SliceCreated', { sliceId: 's1', sliceType: 'SC' }])
    .when('Name Slice', ({ c }) => c.nameSlice('s1', 'Place order'))
    .then(['SliceNamed', { sliceId: 's1', name: 'Place order' }]);
});
```

Each scenario starts a fresh model from its Given events, runs the real command and compares the events it records. Only the fields you name are checked, and `ANY` matches generated ids. `.thenNothing()` is a command that records nothing, and `.thenView(name, read, expected)` checks a read model instead. eventPAD has 62 of these, and the whole run takes about 0.13 seconds.

**The .NET twin** is a package, `ItsyBIT.EventStore.Testing`. It's the same idea for the C# apps (Wishbook, Secret Santa, barrio and the shared API host's modules).

## The convention that makes one harness enough

The .NET apps don't all use the same event store. Some use the ItsyBIT store and some use Marten with Dynamic Consistency Boundaries. A harness that talked to stores would need one adapter per store. This one never sees a store, because every slice is written to the same convention:

- Events, commands and state are plain records.
- A command slice is `public static Decision Decide(IReadOnlyList<object> history, TCommand cmd, …)`. It folds the history into its own state and returns a `Decision`: accepted with events, accepted with nothing, or refused with an error and a kind.
- A view is `Project(history)`.
- The endpoint is the store adapter. It loads the boundary's events, unwraps them to plain objects, calls `Decide`, and appends the result (with tags, for DCB) or returns the refusal.

So decision code has no `IEvent`, no Marten types, and nothing to mock.

## A whole setup: Wishbook

Here is everything Wishbook needs, from the project file to the import, as it stands at commit `f29156c`. The excerpts are trimmed, and the fixtures use neutral values.

**1. The package.** The test project references one package from the private ItsyBIT feed:

```xml
<!-- tests/Wishbook.Tests/Wishbook.Tests.csproj -->
<PackageReference Include="ItsyBIT.EventStore.Testing" Version="0.5.0" />
<PackageReference Include="xunit" Version="2.9.2" />
```

and `nuget.config` at the repo root adds the feed next to nuget.org. The token comes from the environment, never the file:

```xml
<add key="github" value="https://nuget.pkg.github.com/<owner>/index.json" />
…
<add key="ClearTextPassword" value="%GITHUB_TOKEN%" />
```

**2. One global using.** `GlobalUsings.cs` makes the package's types visible everywhere: `global using ItsyBIT.EventStore.Testing;`. Each spec file adds `using static ItsyBIT.EventStore.Testing.Gwt;` for `Event<T>`, `ANY` and friends.

**3. A one-line adapter.** Wishbook has its own `Decision` type from before the package existed, so it tells the harness once how to read one:

```csharp
// tests/Wishbook.Tests/WishbookChapter.cs
public sealed class WishbookChapter(string name) : HistoryChapter<Decision>(name,
    d => d.Accepted ? Outcome.Recorded(d.Events) : Outcome.Refused(d.Error!, d.Kind.ToString()));
```

An app that uses the shared `Decision` doesn't need this file.

**4. Shared fixtures.** `TestParty.cs` holds the Given data every slice's tests start from: a created party, its invite, the ids and a fixed clock. Its header says "use these, don't edit them": a spec that needs more history builds it in its own file.

```csharp
public static class TestParty
{
    public static readonly DateTimeOffset At = new(2026, 10, 9, 12, 0, 0, TimeSpan.Zero);

    public const string PartyId = "p-1";
    public const string ShareId = "s-1";
    public const string OrgKey = "…";   // real-shaped keys, so the no-raw-secrets scan means something
    public static readonly string WisherKey = PartyKeys.DeriveWisherKey(OrgKey);

    public static readonly PartyCreated Created = new(
        PartyId, PartyKinds.Party, PartyKeys.Hash(OrgKey), PartyKeys.Hash(WisherKey), ShareId,
        "Host", 10, "Birthday party", "", At);

    public static readonly InviteUpdated Invite = new(
        PartyId, "Birthday party", "Come and celebrate!", "2026-12-01", "15:00", "18:00",
        "The venue", "https://maps.example/venue", At);
}
```

**5. The spec.** `GuestWishesSpecs.cs` is one feature, so it's one chapter. It names its Given data once, and its `Decide` helper turns Wishbook's argument order (`cmd, history, now`) into the harness's (`history, cmd`) with the clock fixed:

```csharp
public class GuestWishesSpecs
{
    private static readonly WishbookChapter GuestWishes = new("Guest wishes");
    private static readonly DateTimeOffset Now = TestParty.At.AddDays(1);

    private static readonly WishAdded BoardGame = new(TestParty.PartyId, "w-1", "A board game", "", Roles.Organizer, TestParty.At);
    private static readonly WishAdded Kite = new(TestParty.PartyId, "w-2", "A kite", "", Roles.Wisher, TestParty.At);
    private static readonly object[] Party = [TestParty.Created, TestParty.Invite, BoardGame, Kite];

    private static PromiseWishCommand Promise(string wishId, string? by) => new(wishId, by);
    private static Decision Decide(IReadOnlyList<object> h, PromiseWishCommand cmd) => PromiseWish.Decide(cmd, h, Now);
    private static WishPromised Promised(string wishId, string by) => new(TestParty.PartyId, wishId, by, Now);

    [Fact]
    public void A_guest_promises_a_wish() =>
        GuestWishes.Scenario("A guest promises a wish")
            .Given(Party)
            .When("Promise Wish", Promise("w-1", "Guest A"), Decide)
            .Then(Promised("w-1", "Guest A"));

    [Fact]
    public void Another_wish_can_still_be_promised() =>
        GuestWishes.Scenario("Another wish can still be promised")
            .Given(Party, Promised("w-1", "Guest A"))
            .When("Promise Wish", Promise("w-2", "Guest B"), Decide)
            .Then(Event<WishPromised>(new { wishId = "w-2", promisedBy = "Guest B" }));

    [Fact]
    public void Already_promised() =>
        GuestWishes.Scenario("A wish can only be promised once")
            .Given(Party, Promised("w-1", "Guest A"))
            .When("Promise Wish", Promise("w-1", "Guest B"), Decide)
            .ThenNothing(Errors.AlreadyPromised, nameof(ErrorKind.Conflict));

    [Fact]
    public void Removed_wishes_are_gone() =>
        GuestWishes.Scenario("Removed wishes are gone")
            .Given(Party, Promised("w-1", "Guest A"), new WishRemoved(TestParty.PartyId, "w-1", Roles.Organizer, Now))
            .ThenView("Guest Wishes", GuestWishesProjection.Project, new { wishes = new[] { new { wishId = "w-2" } }, open = 1 });
}
```

The first `Then` checks the whole event. The second uses `Event<WishPromised>(new { … })`, which checks only the fields it names. `ThenNothing` is a refusal, with its error and kind. `ThenView` runs the read model's `Project` and checks what it shows. Matching is partial throughout: `ANY` and `Where(…)` cover generated ids and timestamps, and values are compared as camelCase JSON with enums by name. The real file has 28 scenarios: 15 for Promise Wish and 13 for the Guest Wishes view.

**6. The slice.** `src/api/Wishbook/Features/GuestWishes/PromiseWish.cs` is what the test calls. `Decide` is plain: history and a command in, a `Decision` out.

```csharp
public sealed record PromiseWishCommand(string WishId, string? PromisedBy);

public static Decision Decide(PromiseWishCommand cmd, IReadOnlyList<object> history, DateTimeOffset now)
{
    var created = PartyState.Created(history);
    if (created is null) return Decision.NotFound(Errors.NotFound);

    var by = (cmd.PromisedBy ?? "").Trim();
    if (by.Length > Streams.MaxNameLength) return Decision.Invalid(Errors.TooLong);
    if (!PartyState.IsLocked(history)) return Decision.Conflict(LockIn.LockInErrors.NotReady);
    if (!PartyState.LiveWishes(history).Any(w => w.WishId == cmd.WishId && !w.Private))
        return Decision.NotFound(Errors.WishNotFound);
    if (PartyState.Promises(history).ContainsKey(cmd.WishId))
        return Decision.Conflict(Errors.AlreadyPromised);

    return Decision.Accept(new WishPromised(created.PartyId, cmd.WishId, by, now));
}
```

The same file has the endpoint. It resolves the share link, reads the party's stream under its lock, calls `Decide` and appends what it accepted. The test never touches that part.

**7. Running it.**

```sh
GWT_EXPORT=/tmp/x dotnet test tests/Wishbook.Tests
```

That writes one `<File>.test.json` per spec file into `/tmp/x`, which is the next section. You rarely run it by hand: `/release` runs `model_check.py`, which runs the tests the same way, combines the files into `~/eventpad-models/wishbook/latest.json` and compares the result with `docs/slices.json`. That `latest.json` is the file ☰ → Import takes.

## The export

Run the tests with `GWT_EXPORT=<dir>` and each spec file also writes `<File>.test.json`: every passing scenario, with what really happened. `then` is one of `{events}`, `{rejected}`, `{nothing}` or `{view, value}`, and `whenData` is the command's input.

On every deploy, Wishbook, Secret Santa, barrio and the shared host run these tests first, and a small script combines the files into one `latest.json` per app and saves a copy per commit. The combined file is `{app, repo, commit, createdAt, files: [{file, scenarios}]}`. Wishbook's latest has 14 files and 243 scenarios.

Here's the one we'll follow, with the test data swapped for neutral values and two long ids shortened:

```json
{
  "chapter": "Guest wishes",
  "title": "A guest promises a wish",
  "given": [
    { "type": "PartyCreated",  "data": { "partyId": "p-1", … } },
    { "type": "InviteUpdated", "data": { "partyId": "p-1", … } },
    { "type": "WishAdded",     "data": { "partyId": "p-1", "wishId": "w-1", "title": "A board game", … } },
    { "type": "WishAdded",     "data": { "partyId": "p-1", "wishId": "w-2", "title": "A kite", … } }
  ],
  "when": "Promise Wish",
  "whenData": { "wishId": "w-1", "promisedBy": "Guest A" },
  "then": { "events": [{ "type": "WishPromised",
    "data": { "partyId": "p-1", "wishId": "w-1", "promisedBy": "Guest A", "promisedAt": "2026-10-10T12:00:00+00:00" } }] },
  "spec":  { "file": "tests/…/GuestWishesSpecs.cs", "line": 29 },
  "slice": { "type": "….GuestWishes.PromiseWish", "method": "Decide",
             "file": "src/…/GuestWishes/PromiseWish.cs", "line": 58 }
}
```

## Where a scenario comes from

The last two keys, `spec` and `slice`, are new in 0.5.0, and they're my build, so here is how they work.

`spec` is the easy one. `Scenario(…)` takes `[CallerFilePath]` and `[CallerLineNumber]`, so the compiler supplies the spec's file and line. The package makes the path repo-relative by walking up to the first folder with `.git` in it, and it strips the `/_/` prefix a deterministic CI build puts on paths.

`slice` is harder. The test only names the command ("Promise Wish"), and the When lambda is compiler-generated, so reflection alone doesn't say which method is the slice. The package reads the lambda's IL instead. .NET has no IL decoder in the box, so it builds an opcode table from `System.Reflection.Emit.OpCodes` once:

```csharp
foreach (var f in typeof(OpCodes).GetFields(BindingFlags.Public | BindingFlags.Static))
{
    if (f.GetValue(null) is not OpCode op) continue;
    var v = (ushort)op.Value;
    if (op.Size == 1) OneByte[v] = op;
    else if ((v & 0xFF00) == 0xFE00) TwoByte[v & 0xFF] = op;
}
```

With that, it steps through the method body, resolves the tokens of every `call`, `callvirt` and `ldftn`, and stops at an unknown opcode rather than misread. It skips calls into the test class, `System.*`, `Microsoft.*` and the package itself, prefers a method named `Decide` (or `Project`, for a view), and follows one level into nested lambdas and spec helpers. Wishbook's spec goes through its `Decide` helper, and that one level is enough to land on `PromiseWish.Decide`.

The file and line come from the portable PDB, read with `System.Reflection.Metadata`: the method's first non-hidden sequence point, cached per assembly. None of it throws. With no PDB you get the type and method only. When the IL can't see the slice, for example a test that goes through an HTTP helper, `.Slice(typeof(X))` names it by hand and wins.

The honest limit: async methods don't get a file yet. Their PDB entry points at the compiler's state machine (`MoveNext`), not the method you wrote. That's why the shared host's chat module has slices with no file. It's planned for 0.6. Today Wishbook, barrio and mycommissions have a code location for every scenario, Secret Santa 98%, and the shared host's modules 88%.

## The naming rule

One rule ties it together: **chapter = feature, When = the exact command name, ThenView = the view name**, the same names `/lets-plan-it` gives the slices in the plan. Then the plan, the tests and the model share one vocabulary, and comparing them needs no mapping table.

## The importer: from export to board

In eventPAD, ☰ → Import takes the file. Everything from here is plain JavaScript in the browser, with zero agent tokens.

**1. What kind of file is it?** `kindOf` in `spec.js` tells a test export apart from eventPAD's own model file (a list of events) and a plan (`docs/slices.json`):

```js
export function kindOf(json) {
  if (json && !Array.isArray(json) && Array.isArray(json.files)) return 'export';
  if (json && !Array.isArray(json) && Array.isArray(json.slices) && json.slices.some(s => s?.kind)) return 'plan';
  if (Array.isArray(json)) {
    if (!json.length) return 'unknown';
    if (json.every(x => x && typeof x === 'object' && 'given' in x && 'then' in x)) return 'scenarios';
    if (json.every(x => x && typeof x === 'object' && typeof x.type === 'string' && ('data' in x || 'id' in x))) return 'events';
    …
```

`scenariosOf` then reads both shapes of `files`: the new `{file, scenarios}` and the older bare lists.

**2. One pass to an outline.** `specFromScenarios` walks the scenarios once and builds plain data. It doesn't have to work out the board's chapters and slices, because the export already carries them:

- **The chapter is written by the test.** Every scenario has a `chapter`. In .NET it's the spec class's chapter field, one per feature: `new HistoryChapter("Guest wishes")`, or in Wishbook `new WishbookChapter("Guest wishes")`; in eventPAD it's `chapter('Slices', …)`. A scenario without one goes into a chapter called "Other".
- **The slice comes from the scenario too.** `when: "Promise Wish"` makes a state-change slice named after that command, and `then.view: "Live Wishes"` makes a view slice named after that read model. Every scenario with the same name in a chapter lands in that one slice.
- **The order is the tests' order.** Chapters come in the order they first appear (the test files by name, then the scenarios in source order), and slices in the order the tests first touch them.

The slice part is four lines:

```js
const isView = !!then.view;
const sliceName = isView ? then.view : s.when;
if (!sliceName) continue;
let slice = ch.slices.find(x => (isView ? x.readModel : x.command) === sliceName);
```

A command slice's events are the ones the tests recorded, not the ones someone expected. Fields are every key that appears in the event data, typed from the value, with any key ending in `Id` typed as a guid. `slice.file:line` becomes the slice's "Code:" note, and each scenario keeps its spec location. All 15 "Promise Wish" scenarios collect into one slice.

**3. Replay through the app's own commands.** `build.js` doesn't write the board directly. It plays the outline through the same commands a tap in the app runs: `createChapter`, `createSlice`, `addToSlice`, then `addScenario`, `setGiven`, `setWhen` and one of `setThenEvent`, `setThenRejection` or `setThenReadModel`. A small `place()` makes sure each name gets one sticky, reused by every slice that mentions it:

```js
const place = (sliceId, type, name, fields) => {
  const k = key(type, name);
  if (ids.has(k)) { c.addToSlice(sliceId, type, { elementId: ids.get(k) }); return ids.get(k); }
  const id = c.addToSlice(sliceId, type, { name });
  ids.set(k, id); addProps(id, fields);
  return id;
};
```

and the scenarios go in the same way a person would add them:

```js
const id = sc.addScenario(sliceId, g.title, g.type, g.source || null);
if (g.given?.length) sc.setGiven(id, g.given.map(x => ({ elementId: ensure('event', x.event, fieldsOf(x.event)), values: x.values })));
if (g.when) sc.setWhen(id, ensure('command', g.when.command, []), g.when.values);
if (g.then?.kind === 'event') sc.setThenEvent(id, ensure('event', g.then.event, fieldsOf(g.then.event)), g.then.values);
```

Because it goes through the real commands, the arrows, the chapter order and the shared stickies come out exactly as the app itself would make them. There's no second idea of what a board looks like to keep in step.

**4. Fold the build away.** For Wishbook that replay is 1,192 events. Their history means nothing (nobody "made" those edits), so the importer folds them into one `ModelSnapshot` event that also carries the repo, commit, date and build. The shelf shows `📸 snapshot @ <sha>`, and any edits you make after that get normal history.

## What lands on the board

Chapter **Guest wishes**, slice **Promise Wish**: the command, then **Wish Promised** with `partyId`, `wishId`, `promisedBy` and `promisedAt`. Under it are 15 scenario cards, each linking to its test line on GitHub at that commit, and the slice's note links to `PromiseWish.cs` line 58. The whole model is 14 chapters, 22 slices, 243 scenarios and 16 event types. The import also reports gaps: events no scenario mentions, events no tested command records, and commands with no refusal case. Wishbook has none of the three.

## What the tests can't say

Some things never appear in a test, so they won't appear on this board:

- **Screens.** A test calls the command; it doesn't know which screen has the button.
- **Automations.** A processor's command looks like any other command from the test's side.
- **Story order and friendly names.** Chapters come in the order of the test files' names, named as the tests name them.

Those come from the plan. The plan can sit on a shelf of its own (outlined in eventPAD, or imported from `docs/slices.json`, which builds a skeleton of its slices), and **Compare shelves** lays the two side by side: only in the plan, only in the code, and slices that differ. Because the names match by rule, that comparison is the whole verification story from yesterday's post, now for free.

Importing never merges into anything. An export always lands on a new shelf of its own; if a shelf with the same name is already there, the summary offers a button to compare with it, and that's all. Connecting the tests to a plan is Compare's job: with the plan as the open board, ⋯ on the imported shelf → **Compare with ‹plan›**. Compare matches slices by their command or read model name (case, spaces and punctuation don't count), so chapter = feature and When = the exact command name is the only mapping plan, tests and model need.

## Where it runs

Every deploy of the four apps runs these tests first and saves the model for that commit. `/release` checks the export against `docs/slices.json` and writes a snapshot into `docs/releases/` in the repo, so every release carries its own model. To look at one, it's ☰ → Import in eventPAD.
