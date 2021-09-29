# Chat Server
## Features
	1. Ephemeral messages. Messages are NOT stored server side, the server merely relays messages to clients
	2. simple protocol
	3. minimal access control (via storing users). To be implemented
	4. tls based encryption. possibly TOFU. To be implemented

## Model
	### Server:
		- socket // the server socket
		- clients // a list of clients, their file descriptors
		- message_queue // a queue of messages, a message contains 1. a tick no. (filled by server), 2. a message body., 3. a sender field (filled by server)
				// 4. a timestamp (filled by client).

A client sent message is defined as follows:
	<8 bits for message_type><64 bits for timestamp><message><nul terminator>
as a result, a real message (translated to ascii/decimal/hex as needed, fields seperated by ---) may look like so:
	0x80---12345678---hello world<nul>

A server sent message is defined as follows:
	<1 bit for is_private><32 bits for server_tick><8 bits for message_count><message1 as defined above><nul terminator><next message>...<nul terminator, 3 times>
as a result, a real message (translated to ascii/decimal/hex as needed, fields seperated by ---) from the server may look like this
	0---1---2---0x80---12345678---hello world<nul><nul>0x80---12345679---dlrow olleh<nul><nul><nul>

A server sent message may also set the is_private bit to true. This is used for server->individual_client. The message format is similiar to the client saent message.

The following message_types are defined. The remaining are reserved for future use.
	senD: send a message
	sigN: register a nick name, the body of the message is the nick to be assigned.
	stoP: sign off politely. Tho body of the message is the message to be forwarded.
	oK  : server accepts a client directive (to set nick, sign off). empty message.
       erroR: server refuses to accept a client directive. Reason in message.
	
Thus, in hex, they become:
	0x44
	0x4e
	0x50
	0x4b
	0x52

An example conversation between client c1, c2, via server s1 can be like so:
	c1	0x4e 12345678 c1\0 \0
	s1	0x4b 12345679 \0 \0
or:
	c1	0x4e 12345678 c1\0 \0
	s1	0x52 12345679 NICK IN USE\0 \0
