---
layout: post
title: "From Tests to Board, With No Agent in Between"
date: 2026-10-25
categories: [tooling]
tags: [event-modeling, eventpad, testing, given-when-then, dotnet, wishbook]
excerpt: "Yesterday's post ended on a hunch: if the tests all share one shape, the model comes out of them with no agents needed. So we followed one Wishbook scenario all the way, from a Given/When/Then test to an eventPAD board."
---

Yesterday we wrote about [reading the model back out of the code]({{ site.baseurl }}{% post_url 2026-10-24-reading-the-model-back-out %}). For barrio that took agents and a couple of million tokens, because its tests come in all sorts of shapes and something had to read each one. We ended on a hunch. If the tests all share **one** shape, a script can read them, and the model just falls out of the test suite.

And now that's how it works. Jocelyn called the walkthrough great content, so we're writing the whole thing down. First the test harness that writes the export, then the importer that turns the export into a board. We'll follow one scenario from Wishbook, a party wishlist app, through both.

## Why tests, and not the docs

Docs and planning boards tell you what someone meant to build. Tests tell you what the code **did**: these events went in, this command ran, and these are the events it actually recorded. If the code disagrees, the test fails. So a model built from passing tests can't drift away from the code, and if the model is wrong, then a test is wrong, which shows up in CI.

But a script can only read tests that all look alike. So the first job was to settle on one shape, with one harness every app can use.

## Two sides of the same coin

There are two implementations of the harness, and they write the same export.

**The JavaScript side** is eventPAD's own, `tests/gwt/gwt.mjs`, in plain Node. The comment at the top of the file shows the shape:

```js
chapter('Slices', () => {
  scenario('Naming a slice')
    .given(['SliceCreated', { sliceId: 's1', sliceType: 'SC' }])
    .when('Name Slice', ({ c }) => c.nameSlice('s1', 'Place order'))
    .then(['SliceNamed', { sliceId: 's1', name: 'Place order' }]);
});
```

Each scenario starts a fresh model from its Given events, runs the real command and compares the events it records. It only checks the fields you name, and `ANY` matches generated ids. `.thenNothing()` is for a command that records nothing, and `.thenView(name, read, expected)` checks a read model instead. eventPAD has 62 of these, and the whole run takes about 0.13 seconds.

**The .NET side** is a package, `ItsyBIT.EventStore.Testing`. Same idea, for the C# apps (Wishbook, Secret Santa, barrio and the shared API host's modules).

## The convention that makes one harness enough

The .NET apps don't all use the same event store. Some use the ItsyBIT store and some use Marten with Dynamic Consistency Boundaries. If the harness talked to the stores, it would need an adapter for each one. It never sees a store, though, because every slice follows the same convention:

- Events, commands and state are plain records.
- A command slice is `public static Decision Decide(IReadOnlyList<object> history, TCommand cmd, …)`. It folds the history into its own state and returns a `Decision`: accepted with events, accepted with nothing, or refused with an error and a kind.
- A view is `Project(history)`.
- The endpoint is the store adapter. It loads the boundary's events, unwraps them to plain objects, calls `Decide`, and appends the result (with tags, for DCB) or returns the refusal.

So the decision code never sees `IEvent` or Marten types, and there's nothing to mock.

## A whole setup: Wishbook

This is everything Wishbook needs, from the project file to the import, as of commit `f29156c`. We trimmed the excerpts, and the fixtures use neutral values.

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

**4. Shared fixtures.** `TestParty.cs` holds the Given data every slice's tests start from: a created party, its invite, the ids and a fixed clock. Its header says "use these, don't edit them", so a spec that needs more history builds it in its own file.

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

The first `Then` checks the whole event. The second uses `Event<WishPromised>(new { … })`, which checks only the fields it names. `ThenNothing` is a refusal, with its error and kind. `ThenView` runs the read model's `Project` and checks what it shows. Matching is partial all the way through. `ANY` and `Where(…)` cover generated ids and timestamps, and values are compared as camelCase JSON, with enums by name. The full file has 28 scenarios, 15 for Promise Wish and 13 for the Guest Wishes view.

**6. The slice.** `src/api/Wishbook/Features/GuestWishes/PromiseWish.cs` is what the test calls. `Decide` takes the history and a command and gives back a `Decision`, nothing else.

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

That writes one `<File>.test.json` per spec file into `/tmp/x` (more on that in the next section). We rarely run it by hand, though. `/release` runs `model_check.py`, which runs the tests the same way, combines the files into `~/eventpad-models/wishbook/latest.json` and compares the result with `docs/slices.json`. That `latest.json` is the file you give ☰ → Import.

## The export

Run the tests with `GWT_EXPORT=<dir>` and each spec file also writes a `<File>.test.json` with every passing scenario and what really happened in it. `then` is one of `{events}`, `{rejected}`, `{nothing}` or `{view, value}`, and `whenData` is the command's input.

On every deploy, Wishbook, Secret Santa, barrio and the shared host run these tests first, and a small script combines the files into one `latest.json` per app and saves a copy per commit. The combined file is `{app, repo, commit, createdAt, files: [{file, scenarios}]}`. Wishbook's latest has 14 files and 243 scenarios.

This is the scenario we're following, with the test data swapped for neutral values and two long ids shortened:

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

The last two keys, `spec` and `slice`, are new in 0.5.0. We built them this week, so a bit more detail on how they work.

`spec` is the easy one. `Scenario(…)` takes `[CallerFilePath]` and `[CallerLineNumber]`, so the compiler supplies the spec's file and line. The package makes the path repo-relative by walking up to the first folder with `.git` in it, and it strips the `/_/` prefix a deterministic CI build puts on paths.

`slice` was trickier. The test only names the command ("Promise Wish"), and the When lambda is compiler-generated, so reflection alone can't tell which method is the slice. Instead, the package reads the lambda's IL. .NET doesn't come with an IL decoder, so it builds an opcode table from `System.Reflection.Emit.OpCodes` once:

```csharp
foreach (var f in typeof(OpCodes).GetFields(BindingFlags.Public | BindingFlags.Static))
{
    if (f.GetValue(null) is not OpCode op) continue;
    var v = (ushort)op.Value;
    if (op.Size == 1) OneByte[v] = op;
    else if ((v & 0xFF00) == 0xFE00) TwoByte[v & 0xFF] = op;
}
```

With that table it steps through the method body and resolves the tokens of every `call`, `callvirt` and `ldftn`. If it hits an opcode it doesn't know, it stops, so it never misreads. It skips calls into the test class, `System.*`, `Microsoft.*` and the package itself, prefers a method named `Decide` (or `Project`, for a view), and follows one level into nested lambdas and spec helpers. Wishbook's spec goes through its own `Decide` helper, and one level is enough to land on `PromiseWish.Decide`.

The file and line come from the portable PDB, read with `System.Reflection.Metadata`. It takes the method's first non-hidden sequence point and caches it per assembly. None of this throws. With no PDB you just get the type and method. And when the IL can't see the slice, say a test that goes through an HTTP helper, you can name it by hand with `.Slice(typeof(X))`, and that wins.

Async methods don't get a file yet, though. Their PDB entry points at the compiler's state machine (`MoveNext`) instead of the method you wrote, which is why the shared host's chat module has slices with no file. That's planned for 0.6. Right now Wishbook, barrio and mycommissions have a code location for every scenario, Secret Santa has 98%, and the shared host's modules 88%.

## The naming rule

All of this hangs on one naming rule. **Chapter = feature, When = the exact command name, ThenView = the view name**, using the same names `/lets-plan-it` gives the slices in the plan. That way the plan, the tests and the model all use the same words, and comparing them needs no mapping table.

## The importer: from export to board

In eventPAD, you give the file to ☰ → Import. From here on it's all JavaScript in the browser, with zero agent tokens.

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

**2. One pass to an outline.** `specFromScenarios` walks the scenarios once and builds an outline as data. It doesn't have to figure out the board's chapters and slices, because the export already has them.

The test writes the chapter. Every scenario has a `chapter`. In .NET it's the spec class's chapter field, one per feature: `new HistoryChapter("Guest wishes")`, or in Wishbook `new WishbookChapter("Guest wishes")`. In eventPAD it's `chapter('Slices', …)`. A scenario without one goes into a chapter called "Other".

The slice comes from the scenario too. `when: "Promise Wish"` makes a state-change slice named after that command, and `then.view: "Live Wishes"` makes a view slice named after that read model. Every scenario with the same name in a chapter lands in that one slice.

And the order is the tests' order. Chapters come in the order they first appear (the test files by name, then the scenarios in source order), and slices in the order the tests first touch them.

The slice part is only four lines:

```js
const isView = !!then.view;
const sliceName = isView ? then.view : s.when;
if (!sliceName) continue;
let slice = ch.slices.find(x => (isView ? x.readModel : x.command) === sliceName);
```

A command slice gets the events the tests actually recorded. Its fields are every key that shows up in the event data, typed from the value, and any key ending in `Id` is a guid. `slice.file:line` becomes the slice's "Code:" note, and each scenario keeps its spec location. All 15 "Promise Wish" scenarios end up in one slice.

**3. Replay through the app's own commands.** `build.js` doesn't write the board directly. It plays the outline through the same commands that run when you tap in the app: `createChapter`, `createSlice`, `addToSlice`, then `addScenario`, `setGiven`, `setWhen` and one of `setThenEvent`, `setThenRejection` or `setThenReadModel`. A small `place()` makes sure each name gets one sticky, reused by every slice that mentions it:

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

Because it goes through the real commands, the arrows, the chapter order and the shared stickies come out just like the app would make them, and we don't have to keep a second copy of the board logic in sync.

**4. Fold the build away.** For Wishbook that replay is 1,192 events. Their history doesn't mean anything (nobody actually "made" those edits), so the importer folds them into one `ModelSnapshot` event that also carries the repo, commit, date and build. The shelf shows `📸 snapshot @ <sha>`, and any edits you make after that get normal history.

## What lands on the board

In chapter **Guest wishes** there's the slice **Promise Wish**, with the command and then **Wish Promised** with `partyId`, `wishId`, `promisedBy` and `promisedAt`. Under it are 15 scenario cards, each linking to its test line on GitHub at that commit, and the slice's note links to line 58 of `PromiseWish.cs`. The whole model is 14 chapters, 22 slices, 243 scenarios and 16 event types. The import also checks for three kinds of gaps: events no scenario mentions, events no tested command records, and commands with no refusal case. Wishbook has none.

## What the tests can't say

Some things never show up in a test, so they won't show up on this board either. Screens are one: a test calls the command, but it doesn't know which screen has the button. Automations are another, because from the test's side a processor's command looks like any other command. And then there's the story order and the friendly names. Chapters come in the order of the test files' names, and they're named the way the tests name them.

Those come from the plan. The plan can sit on a shelf of its own (outlined in eventPAD, or imported from `docs/slices.json`, which builds a skeleton of its slices), and **Compare shelves** puts the two side by side, showing what's only in the plan, what's only in the code, and slices that differ. Since the names match by rule, that comparison does the whole verification from yesterday's post, with no agents.

Importing never merges into anything. An export always lands on a new shelf of its own. If there's already a shelf with the same name, the summary offers a button to compare with it, but that's it. Connecting the tests to a plan is Compare's job. Open the plan as the board, then ⋯ on the imported shelf → **Compare with ‹plan›**. Compare matches slices by their command or read model name (ignoring case, spaces and punctuation), so the naming rule, chapter = feature and When = the exact command name, is the only mapping the plan, the tests and the model need.

## Where it runs

Every deploy of the four apps runs these tests first and saves the model for that commit. `/release` checks the export against `docs/slices.json` and writes a snapshot into `docs/releases/` in the repo, so every release has its own model. If you want to look at one, it's ☰ → Import in eventPAD.
