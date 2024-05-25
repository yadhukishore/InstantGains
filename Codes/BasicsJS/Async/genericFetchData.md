

```javascript
const genericFetchData = async (url) => {
  try {
    const response = await fetch(url);
    const data = await response.json();
    return [data, null]; // Return data and null error
  } catch (error) {
    return [null, error]; // Return null data and error
  }
};

const genericError = (err) => {
  console.error(err);
};

const fetchApiData = async () => {
  const [data, error] = await genericFetchData(
    "https://jsonplaceholder.typicode.com/todos/1"
  );
  if (data) {
    console.log(data);
  } else {
    genericError(error);
  }
};

fetchApiData();
```

### Explanation:

1. **Returning `[data, null]` and `[null, error]`**:
   - Inside `genericFetchData`, if the fetch is successful, return `[data, null]` to indicate that data is present and there's no error.
   - If an error occurs, return `[null, error]` to indicate that there's no data and an error occurred.

2. **Handling the Result in `fetchApiData`**:
   - Destructure the result of `genericFetchData` into `data` and `error`.
   - If `data` is present, log it.
   - If `data` is not present (which means `error` is present), call `genericError` with the error.

### Output:

Now, running the corrected code will give you the expected behavior:

```javascript
{ userId: 1, id: 1, title: 'delectus aut autem', completed: false }
```

This output confirms that `genericFetchData` is successfully fetching the data and returning it properly to be handled in `fetchApiData`.
