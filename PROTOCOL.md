# LoIP Protocol
## Commands (Client)
`\x00 + 32 bytes`: Send SHA-256 ROM checksum to server

`\x01 + 8 bytes`: Connect with other system

`\x02 + 32 bytes`: Send link data to other system (Player 1)

`\x03 + 32 bytes`: Respond to Player 1's link data (Player 2)

`\x04`: Drop connection (P1)

`\x05`: Confirm connection drop (P2)

`\x06 + 4 bytes + n bytes`: Send message, where N is the value of the 4 bytes (P1)

`\x07 + 4 bytes + n bytes`: Send message, where N is the value of the 4 bytes (P2)

## Commands (Server)
`\x00 + 4 bytes`: Comm. error, be sure to display this gameboy style for authenticity ;)

`\x01`: Not yet connected.

`\x02`: Invalid command

`\x03`: Fatal error. Clients should drop the connection.

`\x04`: Command not available.

`\x05`: Reserved

`\x06`: No such player (only in response to `\x01`)

`\x07`: Command data invalid.

`\x08`: Wrong SHA-256 Checksum (only in response to `\x01`)

`\x09`: Response Data ahead.

`\x10`: New link data.

`\x11`: New chat data.

`\x12`: Conn drop request from P1.

`\x13`: Conn drop acceptance from P2.

## Connection Flow
First, you need to send the ROM checksum to the server, then it will respond with a PID (Player ID). Save this in RAM

Second, you need to connect with another system. The server will respond with `\x01` if you are P1, or `\x02` if you are P2.

Third, you need to send link data. **Performing rollback is up to the client**

Lastly, if you are P1, once you are done playing, drop the connection.

If you are P2, you should display a message upon the server sending `\x04`. If P2 accepts, the connection will drop cleanly and the link will be ended.

You can also make calls to `\x06` and `\x07` to use the chat.

## Server connection flow
Upon connection, you should wait for `\x00` to come in. If anything else comes in, respond with `\x04`

Then, once `\x00` comes in, check the next 32 bytes of response data, this will be a SHA-256 checksum. Generate a unique 8 byte ID and return it to the client. 

You'll want to store the SHA-256 checksum with the unique 8 byte ID.

Next, you'll want to wait for `\x01` to come in, anything else, respond with `\x01`.

The next 8 bytes are a unique ID, check if this is in your logged-in players database. If anything else comes in, respond with `\x04`, and if a non-logged in player comes in, respond with `\x06`.

Now, the two systems are connected. **Only `\x04`, `\x05`, `\x06`, and `\x07` should be accepted at this point. Respond with `\x04` to anything else**

Lastly, when you recieve link/chat/conn drop/drop confirm data, send the data to the other player with `\x10`, `\x11`, `\x12`, and `\x13` respectively.
