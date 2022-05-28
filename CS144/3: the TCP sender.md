# Lab Checkpoint 3: the TCP sender

## 1. Overview

The TCPSender is a tool that translates from an outgoing byte stream to segments that will become the payloads of unreliable datagrams.

## 2. Getting started

- Fetch and merg

    ```bash
    git fetch
    git merge origin/lab3-startercode
    ```

## 3. Lab 3: The TCP Sender

`TCPSender`’s responsibility:

- Keep track of the receiver’s window (processing incoming acknos and window sizes)
- Fill the window when possible, by reading from the ByteStream, creating new TCP segments (including SYN and FIN flags if needed), and sending them. The sender
should keep sending segments until either the window is full or the ByteStream is empty.
- Keep track of which segments have been sent but not yet acknowledged by the receiver—we call these “outstanding” segments
- Re-send outstanding segments if enough time passes since they were sent, and they haven’t been acknowledged yet

### 3.1 How does the TCPSender know if a segment was lost?

- Use TCPSender’s `tick` method to timing
- Retransmission timeout (RTO) changs over time but the *initial value* not changes
- Retransmission timer: expires when the RTO has elapsed
- When sending segment, run a timer
- When all outstanding data has been acknowledged, stop the retransmission timer
- When the retransmission timer has expired:
  - Retransmit the earliest egment that hasn’t been fully acknowledged by the TCP receiver
  - If the windows size is nonzero: increment the number of consecutive retransmissions
  - Double the value of RTO.
- When ackno renew
  - Set RTO to *initial value*
  - Rest timer if have any outstanding data
  - Set *consecutive retransmissions* to zero

### 3.2 Implementing the TCP sender

### 3.3 Theory of testing

### 3.4 FAQs and special cases
