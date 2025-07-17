JS Shell Emulator
==========

JS Shell Emulator is a dead simple pure JavaScript library for emulating a shell environment.

[LIVE DEMO](https://francoisburdy.github.io/js-shell-emulator/demos/cli.html)

<img src="https://github.com/francoisburdy/js-shell-emulator/raw/master/demos/screenshot.png" width="636">

This package is inspired from `eosterberg/terminaljs` but written with async/await functions and features enriched.

## Install

```bash
# Using NPM
npm install js-shell-emulator

# Using Yarn
yarn add js-shell-emulator
```

## Get started

Create a container element with an id.

```html

<div id="container"></div>
```

Import module and create a JsShell instance.

```js
import { JsShell } from "js-shell-emulator";

let shell = new JsShell("#container")
shell.print('Hello, world!')
```

## Methods

### Display text & content

```js
// prints a text line
shell.print("Print this message")

// prints a rich html text
shell.printHTML("<strong>Print this bold message</strong>")

// prints a piece of text without line break at the end
shell.write("Print this")

// progressive display, simulates typing. Prints a char every 50ms 
shell.type("This will be displayed gradually", 50)

// prints a piece of text without line break at the end
shell.write("Print this ")
  .write("message")
  .newLine()
```

### Prompt for input

```js

// Ask for text
let name = await shell.input("What's your name?")

// Ask for a secret, input won't be shown during typing.
let secret = await shell.password("Enter your password")

// Ask for confirmation. "(y/n)" will be append at the end of the question. 
let confirm = await shell.confirm("Are you sure?")
```

### Interface customization

#### Constructor options

Below are the (default) styling options that you can pass in the constructor second parameter.

```js
const shell = new JsShell('#container', {
  backgroundColor: '#000',
  textColor: '#fff',
  className: 'jsShell', // this class will be applied on the shell root element.
  cursorType: 'large', // Typing cursor style: "large" ▯ or "thin" |
  cursorSpeed: 500, // blinking interval in ms
  fontFamily: 'Ubuntu Mono, Monaco, Courier, monospace',
  forceFocus: false, // whether or not inputs should capture document focus even if active element is outside the shell
  textSize: '1em',
  promptPS: '', // Prompt PS1 prefix ($, #, > or whatever you like) 
  width: '100%', // Shell root element css width
  height: '300px', // Shell root element css height
  margin: '0',
  overflow: 'auto',
  whiteSpace: 'break-spaces',
  padding: '10px',
})
```

#### Dynamic setters

You can programatically update styles using the follow setters:

```js
 shell
  .setTextSize('0.9rem')
  .setTextColor('green')
  .setFontFamily('consolas')
  .setForceFocus(true)
  .setBackgroundColor('#000')
  .setWidth('100%')
  .setHeight('400px')
  .setMargin('10px auto')
  .setBlinking(true) // start or stop cursor blinking
  .setPrompt('$ ')
  .setVisible(true)  // show or hide terminal
```

#### Custom CSS

The package is CSS free, but you can apply any other styles on the root terminal class:

```css
.jsShell {
    opacity: 0.9;
    line-height: 120%;
}
```

### Play time

```js
// Wait a second
await JsShell.sleep(1000)

// Give user a break
await shell.pause("Press any key to continue.")
```

### Other methods

```js
// Clear the terminal screen
shell.clear()

// Focus the shell prompt
shell.focus()

// Update current input programmatically (used with key handlers)
shell.updateInput('new value', inputField)

// Set a universal key handler for advanced input handling
shell.setKeyHandler((keyEvent, shellInstance) => {
  // Handle key events with full control
  // Return string to update input, true to prevent default, false for default behavior
  if (keyEvent.ctrlKey && keyEvent.code === 'KeyL') {
    shellInstance.clear()
    return true // Prevent default behavior
  }
  return false // Use default behavior
})
```

### Advanced Input Handling with `setKeyHandler`

The `setKeyHandler` method allows you to implement advanced keyboard functionality like command history, tab completion, and custom shortcuts. The handler receives a `keyEvent` object and the shell instance, and can return different values to control behavior:

- **Return a string**: Updates the input field with that string
- **Return `true`**: Prevents default behavior (event handled)
- **Return `false`**: Uses default behavior

#### Command History Example

Here's a complete example showing how to implement command history navigation with arrow keys:

```js
// Command history storage
const commandHistory = [];
let historyIndex = -1;

// Set up the key handler
shell.setKeyHandler((keyEvent, shellInstance) => {
  const { code, currentInput } = keyEvent;

  // Arrow Up - navigate to previous command
  if (code === 'ArrowUp') {
    if (commandHistory.length > 0) {
      historyIndex = Math.min(historyIndex + 1, commandHistory.length - 1);
      const command = commandHistory[commandHistory.length - 1 - historyIndex];
      return command; // Update input with historical command
    }
    return true; // Prevent default
  }

  // Arrow Down - navigate to next command
  if (code === 'ArrowDown') {
    if (historyIndex > -1) {
      historyIndex = Math.max(historyIndex - 1, -1);
      if (historyIndex === -1) {
        return ''; // Clear input when at the end
      }
      const command = commandHistory[commandHistory.length - 1 - historyIndex];
      return command;
    }
    return true; // Prevent default
  }

  // Ctrl+L - clear screen
  if (keyEvent.ctrlKey && code === 'KeyL') {
    shellInstance.clear();
    return true;
  }

  return false; // Use default behavior for other keys
});

// Main terminal loop with history management
async function terminalLoop() {
  while (true) {
    const input = await shell.input('$ ');
    
    // Add non-empty commands to history
    if (input.trim() && input.trim() !== commandHistory[commandHistory.length - 1]) {
      commandHistory.push(input.trim());
    }
    
    // Reset history index
    historyIndex = -1;
    
    // Process command
    if (input.trim() === 'exit') break;
    shell.print(`You entered: ${input}`);
  }
}
```

#### KeyEvent Object Properties

The `keyEvent` object passed to your handler contains:

```js
{
  key: string,           // The key value (e.g., 'a', 'Enter', 'ArrowUp')
  code: string,          // The key code (e.g., 'KeyA', 'Enter', 'ArrowUp')
  keyCode: number,       // Legacy key code
  ctrlKey: boolean,      // True if Ctrl key is pressed
  shiftKey: boolean,     // True if Shift key is pressed
  altKey: boolean,       // True if Alt key is pressed
  metaKey: boolean,      // True if Meta/Cmd key is pressed
  currentInput: string,  // Current input field value
  promptType: number,    // Type of prompt (INPUT, PASSWORD, CONFIRM, PAUSE)
  preventDefault: function, // Call to prevent default behavior
  stopPropagation: function // Call to stop event propagation
}
```

#### Additional Examples

```js
// Tab completion
shell.setKeyHandler((keyEvent) => {
  if (keyEvent.code === 'Tab') {
    const commands = ['help', 'list', 'clear', 'exit'];
    const matches = commands.filter(cmd => cmd.startsWith(keyEvent.currentInput));
    if (matches.length === 1) {
      return matches[0] + ' '; // Auto-complete and add space
    }
    return true; // Prevent default tab behavior
  }
  return false;
});

// Custom shortcuts
shell.setKeyHandler((keyEvent, shellInstance) => {
  // Ctrl+D for quick exit
  if (keyEvent.ctrlKey && keyEvent.code === 'KeyD') {
    if (keyEvent.currentInput === '') {
      shellInstance.print('exit');
      return true;
    }
  }
  
  // Ctrl+U to clear current line
  if (keyEvent.ctrlKey && keyEvent.code === 'KeyU') {
    return ''; // Clear input
  }
  
  return false;
});
```

## License

See [LICENSE.md](LICENSE.md)
