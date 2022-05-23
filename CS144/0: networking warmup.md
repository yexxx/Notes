# Lab Checkpoint 0: networking warmup

## 1. Set up GNU/Linux on your computer

What I use is Microsoft WSL2.

## 2. Networking by hand

- Fetch a Web page

    ```bash
    telnet cs144.keithw.org http
    GET /lab0/test HTTP/1.1
    Host: cs144.keithw.org
    Connection: close
    ```

- Listening and connecting

  - Terminal 1: `netcat -v -l -p 9090`
  - Terminal 2: `telnet localhost 9090`

## 3. Writing a network program using an OS stream socket

- With "best-effort datagram", datagrams can be:
  - lost,
  - delivered out of order,
  - delivered with the contents altered, or even
  - duplicated and delivered more than once.

- Source code: `git clone https://github.com/cs144/sponge`

- SPECIFICATION
  - Use the language documentation at [cppreference](https://en.cppreference.com) as a resource.
  - Never use malloc() or free().
  - Never use new or delete.
  - Essentially never use raw pointers (*), and use “smart” pointers (unique ptr or shared ptr) only when necessary. (You will not need to use these in CS144.)
  - Avoid templates, threads, locks, and virtual functions. (You will not need to use these in CS144.)
  - Avoid C-style strings (char *str) or string functions (strlen(), strcpy()). These are pretty error-prone. Use a std::string instead.
  - Never use C-style casts (e.g., (FILE *)x). Use a C++ static cast if you have to (you generally will not need this in CS144).
  - Prefer passing function arguments by const reference (e.g.: const Address & address).
  - Make every variable const unless it needs to be mutated.
  - Make every method const unless it needs to mutate the object.
  - Avoid global variables, and give every variable the smallest scope possible.
  - Before handing in an assignment, please run make format to normalize the coding style.

- Read the interface in the libsponge/util directory: file descriptor.hh, socket.hh, and address.hh.
