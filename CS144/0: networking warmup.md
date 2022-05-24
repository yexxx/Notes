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

- Read the interface [file descriptor](https://cs144.github.io/doc/lab0/class_file_descriptor.html), [socket](https://cs144.github.io/doc/lab0/class_socket.html), [address](https://cs144.github.io/doc/lab0/class_address.html) in the libsponge/util directory: file descriptor.hh, socket.hh, and address.hh.

- Writing webget:
  - Create `TCPSocket` -> `connect` -> `write` -> `shutdown` -> `read` -> `close`
  - Each line must be ended with `"\r\n"` and eventually end with `"\r\n"`

  ```cpp
    TCPSocket tcpsocket{};
    tcpsocket.connect(Address(host, "http"));
    tcpsocket.write("GET " + path + " HTTP/1.1\r\n" + "Host: " + host + "\r\n" + "Connection: close\r\n\r\n");
    tcpsocket.shutdown(SHUT_WR);
    while (!tcpsocket.eof()) {
        cout << tcpsocket.read();
    }
    tcpsocket.close();
  ```

## 4. An in-memory reliable byte stream

- Data structure

  ```cpp
    size_t _capacity{};
    std::deque<char> buf{};
    long long _bytes_read{0}, _bytes_written{0};
    bool _end_input{false};
  ```

- Init and some control function

  ```cpp
  ByteStream::ByteStream(const size_t capacity) : _capacity{capacity} {}

  void ByteStream::end_input() { _end_input = true; }

  bool ByteStream::input_ended() const { return _end_input; }

  size_t ByteStream::buffer_size() const { return buf.size(); }

  bool ByteStream::buffer_empty() const { return buf.empty(); }

  bool ByteStream::eof() const { return buf.empty() && _end_input; }

  size_t ByteStream::bytes_written() const { return _bytes_written; }

  size_t ByteStream::bytes_read() const { return _bytes_read; }

  size_t ByteStream::remaining_capacity() const { return _capacity - buf.size(); }
  ```

- Write and read(peek and pop)

  ```cpp
  size_t ByteStream::write(const string &data) {
    size_t can_write = std::min(data.size(), remaining_capacity());
    for (size_t i = 0; i < can_write; ++i)
        buf.push_back(data[i]);
    _bytes_written += can_write;
    return can_write;
  }

  string ByteStream::peek_output(const size_t len) const {
      return string(buf.begin(), buf.begin() + std::min(len, buffer_size()));
  }

  void ByteStream::pop_output(const size_t len) {
      _bytes_read += std::min(len, buffer_size());
      if (len > buffer_size())
          deque<char>{}.swap(buf);
      else
          for (size_t i = 0; i < len; ++i)
              buf.pop_front();
  }

  std::string ByteStream::read(const size_t len) {
      string read_string = peek_output(len);
      pop_output(len);
      return read_string;
  }
  ```
