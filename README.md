# Browser extensions for netcode.io

This repository enables the use of [netcode.io](https://github.com/networkprotocol/netcode.io) via browser extensions, prior to it's adoption in browsers.

**This is super experimental right now!**

![gif of browser support](https://media.giphy.com/media/100PoL7yGm4Fi0/giphy.gif)

## Installation

For this to work, it requires the installation of both a browser extension and a native application helper which performs the actual netcode.io communication.  The extension uses the native messaging APIs provided in browsers in order to make netcode.io available via the helper.

To try this out in Google Chrome, you'll first need to build the `netcode.io.host.sln` solution in Visual Studio, then run `Install-Windows.ps1` in Powershell, which will install the netcode.io helper into the Windows registry.

After this is done, add the `browser\chrome` directory as an unpacked extension.

The intention is that in the future, the extension will be available in the Chrome Web Store (and extension stores for Edge and Firefox). Upon installation of the extension, it would provide a native installation executable that sets up the helper for the user.

## Supported Browsers

So far this has been tested in Google Chrome, though it should translate to Edge and Firefox easily via the new WebExtensions specification.

Mobile device support is basically impossible until netcode.io support appears in browsers natively, as mobile platforms don't support extensions or native messaging.

## netcode.io API

The API made available to Javascript clients as `window.netcode`. You can check for the availability of netcode.io using the conventional `if (window.netcode) { ... }` pattern.

`window.netcode` provides one function: `createClient(callback)`:

### window.netcode.createClient(callback)

Creates a new netcode.io client which can be used for secure UDP communication using the netcode.io protocol. The callback is of the form `callback(err, client)`. If `err` is set, there was an error creating the client (and `client` will be null). Otherwise `client` is set and `err` is null. The returned client is an instance of `Client`.

**Parameters:**
- `callback`: A callback in the form `callback(err, client)` where `err` is either `null` or an instance of `Error`, and `client` is either `null` or an instance of `Client`.

### Client.setTickRate(tickRate, callback)

Sets the tick rate of the netcode.io client, expressed as the number of ticks per second for receiving and sending packets.  The tick rate for clients defaults to `60`; that is, 60 times a second.

**Parameters:**
- `tickRate`: An integer that is equal to or greater than `1`.
- `callback`: A callback in the form `callback(err)` where `err` is either `null` or an instance of `Error`.

### Client.connect(token, callback)

Connects to a netcode.io server using the specified token.  The `token` should be an instance of `Uint8Array` and represent a netcode.io token received from an authentication server.

You can not use netcode.io to send UDP packets to arbitrary IP addresses; instead, you must have an authentication server that uses the netcode.io library (or a compatible implementation) which can generate and sign tokens with a list of game server IP addresses and a private key shared between your authentication server and game servers.

In most common scenarios, your authentication server will provide the token as part of an AJAX request. If you are using a user account system, you'll provide tokens after the user logins into your game, with the token indicating which server the client is authorized to connect to.

**Parameters:**
- `token`: A `Uint8Array` instance which contains the token data.
- `callback`: A callback in the form `callback(err)` where `err` is either `null` or an instance of `Error`.

### Client.send(packetBuffer, callback)

Sends a packet to the connected server with `packetBuffer` as the data. `packetBuffer` should be an instance of `Uint8Array`.

**Parameters:**
- `token`: A `Uint8Array` instance which contains the packet data to send.
- `callback`: A callback in the form `callback(err)` where `err` is either `null` or an instance of `Error`.

### Client.getClientState(callback)

Returns the current state of the client as a string. The returned state is one of: `connected`, `connectionDenied`, `connectionRequestTimeout`, `connectionResponseTimeout`, `connectionTimedOut`, `connectTokenExpired`, `disconnected`, `invalidConnectToken`, `sendingConnectionRequest`, `sendingConnectionResponse` or `destroyed`.

**Parameters:**
- `callback`: A callback in the form `callback(err, state)` where `err` is either `null` or an instance of `Error`. `state` is either `null` (in the case of an error) or one of the states listed above.

### Client.destroy(callback)

Destroys the client, disconnecting it from the server and cleaning up any associated resources. Once a client is destroyed, it can't be reused.

**Parameters:**
- `callback`: A callback in the form `callback(err)` where `err` is either `null` or an instance of `Error`.

### Client.addEventListener(type, callback)

Adds an event listener to the client. Currently the only supported `type` is `receive`, which is fired when the client receives a packet from the server.

For `receive` the callback is of the form `callback(clientId, buffer)` where `clientId` is the client identifier issued originally by the authentication server and `buffer` is the received packet as an instance of `Uint8Array`.

**Parameters:**
- `type`: One of the supported types listed above.
- `callback`: A callback whose form differs based on `type`.

## License

This host extension code is provided under the MIT license.

## Contributing

The primary goals for this project currently are:

- Stabilizing the implementation currently provided here
- Making the extension available in Edge and Firefox
- Making the native helper executable available for Linux and Mac
- Getting the extension into the relevant extension web stores
- Making the installation of the extension and native helper easy enough for players to install it upon prompting from HTML5 games

All pull requests must be made available under an MIT license.