To avoid using `prompt`, which is not available in a Node.js environment or certain non-browser environments, you can use `readline` to get user input in a Node.js environment. Here is the updated code using `readline` for getting user input:

```javascript
const readline = require('readline');

function printPattern(r, c) {
    for (let i = 0; i < r; i++) {
        let rowPattern = '';
        for (let j = 0; j < c; j++) {
            rowPattern += '* ';
        }
        console.log(rowPattern.trim());
    }
}

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

rl.question('Enter the number of rows: ', (rows) => {
    rl.question('Enter the number of columns: ', (cols) => {
        const r = parseInt(rows, 10);
        const c = parseInt(cols, 10);

        if (r > 0 && c > 0) {
            printPattern(r, c);
        } else {
            console.log('Please enter positive values.');
        }

        rl.close();
    });
});
```

In this code:
1. `readline` module is used to read input from the console.
2. The `rl.question` method is used to prompt the user for the number of rows and columns.
3. The inputs are parsed to integers and validated to ensure they are positive.
4. The `printPattern` function is called with the user-provided dimensions if the inputs are valid; otherwise, an error message is displayed.

This approach works well in a Node.js environment and handles user input directly from the command line.
