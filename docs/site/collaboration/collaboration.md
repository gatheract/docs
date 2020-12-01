## Introduction

**GatherAct** has two kinds of apps:

1.  Apps designed to be used in a real-time collaborative way when
    launched inside a virtual platform channel
2.  Apps designed to be used standalone by a single user, these apps are
    not accessible from within a virtual platform

Both forms of apps can utilize the LTI API in order to be launched from
and report to a Learning Management System (LMS)

We make it easy to create these apps by providing these features:

- WebSocket API for real-time collaboration
- REST API for LTI integration
- App discovery in our marketplace
- Payment processing (coming soon)

Join the community on Discord:
[https://discord.gg/QEsktfJ](https://discord.gg/QEsktfJ)

---

## Creating real-time apps

Include the api script:

```html
<script src="https://gatheract.com/api.js"></script>
```

Initialize the api, this will connect the app instance to the server via
a WebSocket

```javascript
gatheract.init({
	appId: 'Enter a unique name or string',
	events: {
		connected: event => {},
		disconnected: event => {},
		channelInfo: event => {},
		appMessage: (data, from) => {}
	}
});
```

When testing you can pass anything to for the _app_id_ eventually this
will need to be the id provided when you register your app

`gatheract.sendMessage` is used to send data between app instances
within the channel. `users` is a list of user ids, if provided the server will only send
this message to instances for those users, otherwise it will broadcast
to all instances

```javascript
let data = { message: 'Hello World' };
gatheract.sendMessage(data);
```

### Interfaces

```typescript
interface IGatheract {
	init: (config: IApiConfig) => void;
	sendMessage: (message: IAppMessage, users?: string[], sendToSelf?: boolean) => void;
	connected: boolean; // indicator if the WebSocket is currently connected to the channel
	appId: string; // the app identifier provided when the api was initialized
	channelId: string; // channel identifier
	users: IUser[]; // list of all users in the channel
	user: IUser; // the user this app instance represents
	host: IUser; // the host is the user that activated the app within the channel
	isHost: boolean; // indicator if the user is the host
}

interface IApiConfig {
	appId: string; // a unique string to identify your app
	events: IAppEvents;
}

interface IAppEvents {
	connected?: (message: IConnected) => void; // callback when the app is connected to a channel
	disconnected?: (message: IDisconnected) => void; // callback when the app is disconnected to a channel
	appMessage?: (data: any, from: string) => void; // callback for messages received from another app instance, from is the id of the from user
	channelInfo?: (message: IChannelInfo) => void; // callback when user is added or removed from channel
	apiMessage?: (message: IMessage) => void; // a catch all message callback
}

interface IUser {
	id: string; // user identifier
	name: string; // display name for that user
}

interface IChannelInfo {
	type: 'channel_info';
	id: string;
	users: IUser[];
	newUser?: IUser;
	leftUser?: IUser;
	host: IUser;
}

interface IConnectedMessage {
	type: 'connected';
	channel_id: string;
	host: IUser; // user id that activated the app
	user: IUser; // the current user users: IUser[];
}

interface IDisconnectedMessage {
	type: 'disconnected';
	channelId: string;
}

interface IAppMessage {
	type: 'app_message';
	data: any; // app defined value or object
	from: string; // from client id
}
```

### Sample App

This simple examples sends a greeting message from the host to users as
they connect

```html
<div class="page" id="create">
	<api-navbar></api-navbar>
	<div class="inner">
		<h2>How to create apps</h2>
		<section>
			<div>
				<p>GatherAct has two kinds of apps:</p>
				<ol>
					<li>
						Apps designed to be used in a real-time collaborative way when launched inside a virtual platform channel
					</li>
					<li>
						Apps designed to be used standalone by a single user, these apps are not accessible from within a virtual
						platform
					</li>
				</ol>
				<div></div>
				<div></div>
			</div>
			<p>
				Both forms of apps can utilize the LTI API in order to be launched from and report to a Learning Management
				System (LMS)
			</p>
			<p>We make it easy to create these apps by providing these features:</p>
			<ul>
				<li>WebSocket API for real-time collaboration</li>
				<li>REST API for LTI integration</li>
				<li>App discovery in our marketplace</li>
				<li>Payment processing (coming soon)</li>
			</ul>
		</section>
		<div style="font-weight: bold; font-size: 1.2em">
			Join the community on Discord:
			<a href="https://discord.gg/QEsktfJ" target="_blank">https://discord.gg/QEsktfJ</a>
		</div>
	</div>
</div>
```

## Platform Integration

## How to integrate GatherAct into your virtual classroom

By integrating **GatherAct** into a virtual platform your users will be able
to launch real time collaborative apps

The first thing to do is create a **GatherAct** account and register your
platform

Once you have a platform id, you can link and initialize our api

include the api script:

```html
<script src="https://gatheract.com/api.js"></script>
```

Then initialize the api:

```javascript
api.init((config: IApiConfig));
```

### Interfaces

```typescript
interface IApiConfig {
	platform_id: string;
	events?: {
		connected?: (message: IConnected) => void;
		disconnected?: (message: IDisconnected) => void;
		channel_info?: (message: IChannelInfo) => void;
		activate_app?: (message: IActivateApp) => void;
		api_message?: (message: IMessage) => void;
	};
}
```
