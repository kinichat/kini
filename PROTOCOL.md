# kiniprotocol

Hi! This is the specification for kini's protocol, where you can find all the details and black magic behind it. Have fun reading it!

## 1. Communication & Packets

kiniprotocol uses TCP and port 15999 for communication. Messages are sent as simple packages, containing the sender's username, message and some additional data for security and data loss prevention:

| Offset(bytes) | Size | Description                                             |
|---------------|------|---------------------------------------------------------|
| 0             | 4    | Package checksum, not including the checksums           |
| 4             | 24   | Username(only ASCII characters, padded with 0x00)       |
| 28            | 4    | Hash of user's password, treated as a signature         |
| 32            | 24   | Channel(only ASCII characters, padded with 0x00)        |
| 56            | 4    | Message length(in bytes): LEN                           |
| 60            | LEN  | Message data                                            |
| 60 + LEN      | 4    | Copy of package checksum, not including the checksums   |

## 2. Hashes and checksums

The checksums in kiniprotocol MUST use the following or an equivalent function for generating hash checksums:

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

The user password hash can be made using any algorithm, but using the same algorithm as used for the hash checksums described above is recommended and should be used by all clients by default.

## 3. Sending & Receiving Messages

Sending messages is as simple as just sending a packet with the structure in section 1 to the server, and then the server broadcasting it to all of the clients connected. Receiving them is really easy too, just wait until you receive a packet with the information in there, do the checks and add it to some temporal array of messages to show on the client.
