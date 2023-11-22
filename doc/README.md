# `cache-server`

A minimal Key/Value store written in Rust. 

This a project by Group#65 in Rust.
Group Members:
- Aarya Aamish Bhatt
- Aastha Bharill
- Arjun Sankholkar
- Ishan Nerlekar

## 1. Problem Statements
- Establishing Fast Data Retrieval: Cache servers like Redis and Memcached offer lightning-fast data access due to in-memory storage, ideal for applications needing swift access to frequently used data.
- Reducing Database Load: Storing frequently accessed data in a cache eases the burden on relational databases, resulting in better performance, quicker responses, and cost savings, especially in read-heavy workloads.
- Caching Complex Queries: Cache servers can store the results of intricate queries, eliminating the need for repetitive calculations and saving valuable processing time.

## 2. Software Architecture
The server follows a multi-threaded architecture where each thread handles a set of connections concurrently. The main function initializes the main server components, including the main listener (TcpListener), main poll (main_poll), and a set of child polls (child_polls). The main listener accepts incoming connections and assigns them to child polls for handling.
### Concurrency and Threading:
The server utilizes multiple threads to handle concurrent connections efficiently. The number of threads is configurable through a command-line argument (threads). Each thread has its event loop represented by a Poll instance.
### Connection Handling:
Each connection is represented by the Conn struct, which includes information about the TCP stream, address, input, output, and other flags. The main_conns (connections on the main thread) and streams (connections on child threads) are managed using HashMap<usize, Conn>. The server employs non-blocking I/O using the mio crate, allowing it to handle multiple connections simultaneously.
### Command Handling:
The handle_command function processes incoming commands, such as SET, GET, DEL, etc. Commands are matched using the arg_match function, and the corresponding logic is executed. The store struct manages the data storage, which contains a HashMap (keys) for key-value pairs.
### Command Execution and Response:
The execution of commands generates a response, which is then sent back to the client. The response is constructed based on the Redis protocol format.
### Asynchronous I/O and Event-driven Design:
The server uses the tokio runtime for asynchronous programming, allowing it to efficiently handle I/O operations without blocking.
### Command Parsing:
Command parsing is implemented in functions like redcon_take_inline_args and redcon_take_multibulk_args, handling both inline and multi-bulk requests.
### Dynamic Configuration:
The server allows dynamic configuration of the number of threads and the listening port through command-line arguments.
### External Dependencies:
External crates, such as mio, num_cpus, clap, and glob, are used to handle asynchronous I/O, obtain the number of CPUs, parse command-line arguments, and perform pattern matching, respectively.
### Error Handling:
Error handling is performed throughout the code using the Result type, ensuring robustness in the face of potential issues.
### Lifetime Management:
Rust lifetime annotations are used where necessary, indicating how long referenced data is valid.
### Visualization:
Consider creating a visual representation of the architecture, showing the main components, their interactions, and the data flow.
### Testing Component:
The code itself does not include explicit testing components. Testing can be done through external tools or by extending the code with test modules.
### Database:
The server includes an in-memory key-value store (Store struct) to manage cached data. No external database integration is present in the provided code. The architecture follows a multi-threaded, event-driven model emphasising asynchronous I/O and efficient connection handling. The server manages a simple in-memory key-value store to handle Redis-like commands.

## 3. POPL Aspects
Implementation Highlights:
List 5 to 10 key POPL aspects involved in the implementation. Provide specific pointers to lines of code and explain the POPL ideas or concepts incorporated. Share your experience regarding any difficulties faced during implementation.

### Aspect 1: Ownership and Borrowing:
Arc (Atomic Reference Counting)

Pointer to Code: [Ownership and Borrowing](https://github.com/ishan-nerlekar/Cache-Server-POPL/blob/main/src/main.rs#L85)
- Explanation: main_conns and store are declared as Arc<Mutex<HashMap<usize, Conn>>> and Arc<Mutex<Store>>, respectively. Arc provides shared ownership over its content, and Mutex helps manage concurrent mutable access to the shared data.

### Aspect 2: References:
Function Parameters

Pointer to Code: [References](https://github.com/ishan-nerlekar/Cache-Server-POPL/blob/main/src/main.rs#L455)
- Explanation: Functions like event_data, handle_command, and others take references (&) to arguments instead of taking ownership of them. This allows them to borrow data for the duration of the function call without taking full ownership. For example, fn event_data(..., store: &Arc<Mutex<Store>>).

### Aspect 3: Shadowing:
Pointer to Code: [Shadowing](https://github.com/ishan-nerlekar/Cache-Server-POPL/blob/main/src/main.rs#L59)
- Explanation: In this code snippet, the variable 'threads' is shadowed when it is redefined with a new value. It initially holds the parsed value of the command-line argument, and if the value is not present or invalid, it takes on the value returned by num_cpus::get()

### Aspect 4: Dangling Pointers:
Pointer to Code: [Dangling Pointers](https://github.com/ishan-nerlekar/Cache-Server-POPL/blob/main/src/main.rs#L493)
- Explanation: In the event_data function, this code removes processed data from the input vector. It ensures that if i is within the valid bounds of the vector, the data is cleared and, if needed, transferred to a new vector (remain). This approach avoids leaving any dangling pointers to the processed data.

## 4. Results
Test Results:
Present the test results, datasets used, and any benchmarks run. Include visual representations such as line graphs or bar graphs to showcase performance. Explain the methodology for checking and validating results aligning with the initial problem statement. Provide data-driven proof points to convince users that the system is working.

## 5. Potential for Future Work
Future Development:
Outline potential areas for future work. Discuss what additional POPL aspects might come into play given more time. Share your vision for expanding the project and improving its functionality.


