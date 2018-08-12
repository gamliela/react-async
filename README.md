# `<Async>`

React component for declarative promise resolution and data fetching. Uses render props for full flexibility or React context
for convenience. Makes it easy to handle loading and error states, without assumptions about the shape of your data or the type of request. Supports optimistic updates.

## Install

`npm install --save react-async`

## Usage

Using render props for ultimate flexibility:

```js
import Async from "react-async"

const loadJson = () => fetch("/some/url").then(res => res.json())

const MyComponent = () => {
  return (
    <Async promiseFn={loadJson}>
      {({ data, error, isLoading }) => {
        if (isLoading) return "Loading..."
        if (error) return `Something went wrong: ${error.message}`
        if (data)
          return (
            <div>
              <strong>Loaded some data:</strong>
              <pre>{JSON.stringify(data, null, 2)}</pre>
            </div>
          )
        return null
      }}
    </Async>
  )
}
```

Using helper components (don't have to be direct children) for ease of use:

```js
import Async from "react-async"

const loadJson = () => fetch("/some/url").then(res => res.json())

const MyComponent = () => {
  return (
    <Async promiseFn={loadJson}>
      <Async.Loading>Loading...</Async.Loading>
      <Async.Resolved>
        {data => (
          <div>
            <strong>Loaded some data:</strong>
            <pre>{JSON.stringify(data, null, 2)}</pre>
          </div>
        )}
      </Async.Resolved>
      <Async.Rejected>{error => `Something went wrong: ${error.message}`}</Async.Rejected>
    </Async>
  )
}
```

`<Async>` is promise-based, so you can resolve anything you want, not just `fetch` requests.

### Props

`<Async>` takes the following properties:

- `promiseFn` {() => Promise} A function that returns a promise; invoked immediately in `componentDidMount` and without arguments
- `deferFn` {() => Promise} A function that returns a promise; invoked only by calling `run`, with arguments being passed through
- `watch` {any} Watches this property through `componentDidUpdate` and re-runs the `promiseFn` when the value changes (`oldValue !== newValue`)
- `onResolve` {Function} Callback function invoked when a promise resolves, receives data as argument
- `onReject` {Function} Callback function invoked when a promise rejects, receives error as argument

### Render props

`<Async>` provides the following render props:

- `data` {any} last resolved promise value, maintained when new error arrives
- `error` {Error} rejected promise reason, cleared when new data arrives
- `isLoading` {boolean} `true` while a promise is pending
- `startedAt` {Date} when the current/last promise was started
- `finishedAt` {Date} when the last promise was resolved or rejected
- `cancel` {Function} ignores the result of the currently pending promise
- `run` {Function} runs the `deferFn`, passing any arguments provided
- `reload` {Function} re-runs the promise when invoked, using the previous arguments
- `setData` {Function} sets `data` to the passed value, unsets `error` and cancels any pending promise
- `setError` {Function} sets `error` to the passed value and cancels any pending promise

## Examples

### Basic data fetching with loading indicator, error state and retry

```js
<Async promiseFn={loadJson}>
  {({ data, error, isLoading, reload }) => {
    if (isLoading) {
      return <div>Loading...</div>
    }
    if (error) {
      return (
        <div>
          <p>{error.toString()}</p>
          <button onClick={reload}>try again</button>
        </div>
      )
    }
    if (data) {
      return <pre>{JSON.stringify(data, null, 2)}</pre>
    }
    return null
  }}
</Async>
```

### Using `deferFn` to trigger an update (e.g. POST / PUT request)

```js
<Async deferFn={subscribeToNewsletter}>
  {({ error, isLoading, run }) => (
    <form onSubmit={run}>
      <input type="email" name="email" />
      <button type="submit" disabled={isLoading}>
        Subscribe
      </button>
      {error && <p>{error.toString()}</p>}
    </form>
  )}
</Async>
```

### Using both `promiseFn` and `deferFn` along with `setData` to implement optimistic updates

```js
const updateAttendance = attend => fetch(...).then(() => attend, () => !attend)

<Async promiseFn={getAttendance} deferFn={updateAttendance}>
  {({ data: isAttending, isLoading, run, setData }) => (
    <Toggle
      on={isAttending}
      onClick={() => {
        setData(!isAttending)
        run(!isAttending)
      }}
      disabled={isLoading}
    />
  )}
</Async>
```

## Helper components

`Async` provides several helper components that make your JSX even more declarative.
They don't have to be direct children of `<Async>` and you can use the same component several times.

### `<Async.Loading>`

Renders only while the promise is still pending.

#### Props

- `initial` {boolean} Show only on initial load (data is undefined)
- `children` {Function|Node} Function which receives props object or React node

#### Examples

```js
<Async.Loading initial>
  <p>This text is only rendered while performing the initial load.</p>
</Async.Loading>
```

```js
<Async.Loading>{({ startedAt }) => `Loading since ${startedAt.toISOString()}`}</Async.Loading>
```

### `<Async.Resolved>`

Renders only when the promise is resolved.

#### Props

- `persist` {boolean} Show old data while loading new data. By default it hides as soon as a new promise starts.
- `children` {Function|Node} Render function which receives data and props object or just a plain React node.

#### Examples

```js
<Async.Resolved persist>{({ data }) => <pre>{JSON.stringify(data)}</pre>}</Async.Resolved>
```

```js
<Async.Resolved>{({ finishedAt }) => `Last updated ${startedAt.toISOString()}`}</Async.Resolved>
```

### `<Async.Rejected>`

Renders only when the promise is rejected.

#### Props

- `persist` {boolean} Show old error while loading new data. By default it hides as soon as a new promise starts.
- `children` {Function|Node} Render function which receives error and props object or just a plain React node.

#### Examples

```js
<Async.Rejected persist>Oops.</Async.Rejected>
```

```js
<Async.Rejected>{({ error }) => `Unexpected error: ${error.message}`}</Async.Rejected>
```
