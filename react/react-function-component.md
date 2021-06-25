React arrow function should be used more often than just using plain functions.

This:

```jsx
const App = () => {
  const greeting = "Hello Function Component!";

  return <Headline value={greeting} />;
};
```

over this:

```jsx
function App() {
  const greeting = "Hello Function Component!";

  return <Headline value={greeting} />;
}
```

Try to use destructuring

```jsx
const Headline = ({ value }) => <h1>{value}</h1>;
```
