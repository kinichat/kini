# kiniprotocol

Hi! This is the specification for kini's protocol, where you can find all the details and black magic behind it. Have fun reading it!

## 1. Communication & Packets

kiniprotocol uses a TCP-based communication library called [msgbox](https://github.com/tylerneylon/msgbox) and port 15999 for communication. Messages are sent as simple packages, containing the sender's username, message and some additional data for security and data loss prevention:

Packet sent by the sender:

| Offset(bytes) | Size | Description                                             |
|---------------|------|---------------------------------------------------------|
| 0             | 4    | Package checksum, not including the checksum            |
| 4             | 4    | User's password, as a 32-bit unsigned integer           |
| 8             | 4    | Message length(in bytes): LEN                           |
| 12            | 20   | Username(NULL-terminated)                               |
| 32            | 32   | Channel(NULL-terminated)                                |
| 64            | LEN  | Message data                                            |

Packet broadcasted by the server(replaces password with hash):

| Offset(bytes) | Size | Description                                             |
|---------------|------|---------------------------------------------------------|
| 0             | 4    | Package checksum, not including the checksum            |
| 4             | 4    | Hash of user's password, treated as a signature         |
| 8             | 4    | Message length(in bytes): LEN                           |
| 12            | 20   | Username(NULL-terminated)                               |
| 32            | 32   | Channel(NULL-terminated)                                |
| 64            | LEN  | Message data                                            |

## 2. Hashes and checksums

The checksums and string hashes in kiniprotocol MUST use the following or an equivalent function for generating hashes from strings:

```c

uint32_t wait_wat(const void *data, uint32_t length) {
  uint32_t x = 0xfb73c5fc;
  const uint8_t *buffer = data;

  while (length) {
    x += buffer[--length];
    x ^= x >> 15;
    x *= 0x2c1b3c6d;
    x ^= x >> 12;
    x *= 0x297a2d39;
    x ^= x >> 15;
  }

  x ^= x >> 17;
  x *= 0xed5ad4bb;
  x ^= x >> 11;
  x *= 0xac4c1b51;
  x ^= x >> 15;
  x *= 0x31848bab;
  x ^= x >> 14;

  return x + 0xefa96b94;
}

```

Additionaly, the 32-bit password hashing on the server MUST be done using this or an equivalent function:

```c

uint32_t oh_fuck(uint32_t x) {
  x ^= x >> 17;
  x *= 0xed5ad4bb;
  x ^= x >> 11;
  x *= 0xac4c1b51;
  x ^= x >> 15;
  x *= 0x31848bab;
  x ^= x >> 14;

  return x;
}

```

## 3. Sending & Receiving Messages

### On connection

1. The new client sends a message containing it's username and password(32-bit unsigned integer) with an empty string as a channel to the server(the contents don't matter).
2. The server sends a message with any username and password hash, but containing a newline-separated list of channels as the content and sent through that same channel, but only to the client that sent the message, not broadcasted.

### After connection

1. The sender sends the message containing it's password(32-bit unsigned integer) to the server.
2. The server broadcasts the message but replacing the password with a hash of it taken with the function above.
3. The clients receive the message.
4. Everyone's happy!

