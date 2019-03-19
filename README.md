# React Knowledge Base

In this repository I will attempt to collect some links and notes relating to React and React + TypeScript.
I keep coming across interesting things in this area, but never write anything down unless I create a PoC to demonstrate something and
put that up on GitHub myself. I usually fail to search for interesting stuff I haven't put up myself, because I forgot the main keywords.

## `React.SFC` and `React.FunctionComponent` returning `string`

I found that `SFC` and `FunctionComponent` cannot return a `string` because in the typings they say they return `ReactElement`.
To be able to do this, they would have to return `ReactChild` or `ReactNode`.
They cannot return `ReactNode` due to some TypeScript compiler limitations described in https://github.com/DefinitelyTyped/DefinitelyTyped/issues/18912.
I think they might be able to return `ReactChild` so I have commented on that issue and if I hear back, will PR and update this note.

## Preventing `class` components without state from having `setState` called

For the longest time I thought I had to do `<ComponentProps, never>` so that calling `setState` in a class component fails.
I found that the default for the state `S` generic type argument is `{}` and when I thought I'd PR that to be `never` instead,
I decided to check my assumptions and found that `setState` can actually still be called, just not with any parameters.
This is meaningfully true for the current default, `{}`, as well, because calling `setState` with `{}` is a no-op and nothing else
can be passed in with the empty object literal default that we have today.

## Using the updater function form of `setState` with conditional logic returning different state objects in different branches

Before `Pick` was introduced in TypeScript, `setState` used to accept `Partial`. This was a nice solution, as it allowed to pass in
partial state updates, any missing keys would just `undefined` and the reconciliation algorithm would be able to deal with that.
Unfortunately, it also allowed passing in state update objects like `{ stringField: undefined }` because that is a valid instance of
`Partial` even though the original type doesn't allow `undefined` on the field `stringField`.

This is undesirable and with `Pick` no longer an issue, because `Pick` will understand that you are passing in a partial state update
object and instead of pretending it is a full object with `undefined` values where not explicitly given like `Partial` does, this
wouldn't allow passing explicit `undefined` to fields that do not allow it.

But due to some TypeScript compiler limitations, `Pick` is not able to understand state updates like these:

```typescript
setState(state => {
  if (state.flag) return { flagUpdate: new Date() };
  return { nonFlagUpdate: new Date() };
})
```

More at https://github.com/DefinitelyTyped/DefinitelyTyped/issues/33697
