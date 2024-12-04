# Signal
A programming language
## Signal Scope
Signals notify all potential listeners in the same scope as the signal.

This means that signals are not global and are only visible to the same scope in which they are defined.

## Types
### Basic Types
int, float, string, bool, signal, error, none
### Complex Types
array, map
### Composite Types
Type unions can be used to assemble multiple basic types into a single type. `int|float|none` 

Type unions can also be used to assemble arrays or maps of multiple types. `array[int|float]` or `map[string, int|float]`

Type unions are also how errors are returned from functions. `int|DivideByZeroError`

### Units
Units are a special type that can be instantiated and used in the program.

They are groups of variables, functions, events, and handlers that are grouped together for common functionality.

### Example
    int a = 5;
    float b = 3.14;
    string c = "Hello, World!";
    bool d = true;
    signal e;

    array[int] f = new array[int];
    map[string, int] g = new map[string, int];

    unit MyUnit {
        local int x;
        local float y;
        #create(self) {
            self.x = 10;
            self.y = 3.14;
        }
    }

## Functions and Hooks
Funcitons can be declared with `fn` and can have event hooks attached to them.

Hooks are functions that are declared with `@fn` and are called at specific points in the function's lifecycle.
Hooks can not have their own lifecycle hooks.

## Errors
Errors are handed back as type unions and can be matched against. Thrown errors are not allowed.
### Example
    fn sendMessage(string msg, string target) -> str|NetworkError {
        conn = openConnection(target);
        if (conn == none) {
            return NetworkError("Failed to open connection");
        }
        conn.write(msg);
        return "ok";
    }
    var result = write("Hello", "Steve");
    match (result) {
        case str value:
            print("msg result: ", value);
        case NetworkError error:
            print("Error: ", error.message);
    }


## Function hooks
### OnCall
OnCall is triggered before a function is invoked.
### Called
Called is triggered after a function is invoked.
### Example
    fn add(int a, int b) -> int @oncall=onCallHandler @called=callHandler {
        return a + b;
    }

or alternatively

    fn add(int a, int b) -> int {
        return a + b;
    }.on("onCall", onCallHandler).on("called", callHandler);

    @fn onCallHandler(self, args) {
        print("Function ", self.name, " called with arguments ", args);
    }
    @fn callHandler(self, args, retValue) {
        print("Function ", self.name, " called with arguments ", args, " and returned ", retValue);
    }
    int result = add(5, 3);

## Units
Units are groups of variables, functions, events, and handlers that are grouped together for common functionality. Units can be instantiated and used in the program.
### Locals
Local variables are variables that are specific to an instance of a unit.
They are not shared between different instances of the unit.
They are also not accessible outside the unit.
### Events
Events are signals that can be emitted by a unit and listened to by other units.
An event declaration includes its signature, which specifies the types of parameters it accepts as well as the parameters that will be passed to handlers that receive it.
### Example
    unit Logger {
        local string logFilePath; // Path to the log file
        local array[string] buffer; // Buffer to hold log messages before writing to file
        
        // Event for logging messages
        event LogEvent(string message);
        
        // Constructor: Initialize the logger
        @create(self) {
            self.logFilePath = "/path/to/logfile.txt";
            self.buffer = new array[string];
            print("Logger created and initialized.");
        }
        
        // Destructor: Perform cleanup
        @delete(self) {
            // Write remaining buffer to file before deleting
            if (self.buffer.length > 0) {
                writeBufferToFile(self);
            }
            print("Logger cleaned up.");
        }
        
        // Event hook for LogEvent
        onEvent(LogEvent) {
            addToBuffer(self, message);
            if (self.buffer.length >= BUFFER_LIMIT) {
                writeBufferToFile(self);
            }
        }
        
        // Helper function to add a message to the buffer
        private fn addToBuffer(Logger self, string message) {
            self.buffer.push(message);
        }
        
        // Helper function to write buffer to the log file
        private fn writeBufferToFile(Logger self) {
            // Assuming writeFile is a function that writes an array of strings to a file
            writeFile(self.logFilePath, self.buffer);
            self.buffer.clear(); // Clear buffer after writing
        }
    }

    Logger logger = new Logger();
    emit Logger.LogEvent("Log message 1");

Message Client

    unit MessageClient {
        local string name;
        local array[string] chatHistory; // Stores the chat history

        // Event when text is entered by this client's user
        event TextEntered(string text);
        
        // Event when a message is received from the other client
        event MessageReceived(string from, string message);
        
        // Event to send a message
        event SendMessage(string to, string message);
        
        // Constructor: Initialize the MessageClient
        @create(self, string clientName) {
            self.name = clientName;
            self.chatHistory = new array[string];
            print(self.name + " Client created.");
        }
        
        // Handle text entry
        onEvent(TextEntered) {
            addToChatHistory(self, "Me: " + text);
            emit SendMessage("OtherClient", text); // Replace "OtherClient" with actual client name
        }
        
        // Handle receiving a message
        onEvent(MessageReceived) {
            string formattedMessage = from + ": " + message;
            addToChatHistory(self, formattedMessage);
        }
        
        // Helper function to add a message to the chat history
        private fn addToChatHistory(MessageClient self, string message) {
            self.chatHistory.push(message);
        }
    }

    // Example usage
    MessageClient client1 = new MessageClient("Alice");
    MessageClient client2 = new MessageClient("Bob");
    
    // Simulating text entry
    emit client1.TextEntered("Hello, Bob!");
    emit client2.TextEntered("Hi, Alice!");
    
    // Simulating message reception
    emit client2.MessageReceived("Alice", "Hello, Bob!");
    emit client1.MessageReceived("Bob", "Hi, Alice!");

## Special Commands
### WaitFor
The waitfor command blocks advancement of the current execution scope until a specific signal is triggered.
#### Example
    waitfor ProgramExit;
### Emit
The emit command triggers a signal with the specified parameters.
#### Example
    emit FormSubmit("John Smith", "555-234-9165");
