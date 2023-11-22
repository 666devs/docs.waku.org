---
title: Manage Your Filter Subscriptions
---

This guide provides detailed steps to manage [Filter](/overview/concepts/protocols#filter) subscriptions and handle node disconnections in your application. Have a look at the [Filter guide](/guides/js-waku/light-send-receive) for receiving messages with the `Light Push` and `Filter` protocol.

## Overview

Occasionally, your `Filter` subscriptions might disconnect from the Waku Network, resulting in messages not being received by your application. To manage your subscriptions, periodically ping peers to check for an active connection. The error message `"peer has no subscriptions"` indicates a failed ping due to disconnection. You can stop the pings if the disconnection/unsubscription is deliberate.

```mdx-code-block
import FilterPingFlow from "@site/diagrams/_filter-ping-flow.md";

<FilterPingFlow />
```

## Pinging Filter Subscriptions

The `@waku/sdk` package provides a `Filter.ping()` function to ping subscriptions and check for an active connection. To begin, create a `Filter` subscription:

```js
// Create a Filter subscription
const subscription = await node.filter.createSubscription();

// Subscribe to content topics and process new messages
await subscription.subscribe([decoder], callback);
```

Next, create a function to ping and reinitiate the subscription:

```js
const pingAndReinitiateSubscription = async () => {
	try {
		// Ping the subscription
		await subscription.ping();
	} catch (error) {
		if (
			// Check if the error message includes "peer has no subscriptions"
			error instanceof Error &&
			error.message.includes("peer has no subscriptions")
		) {
			// Reinitiate the subscription if the ping fails
			await subscription.subscribe([decoder], callback);
		} else {
			throw error;
		}
	}
};

// Periodically ping the subscription
await pingAndReinitiateSubscription();
```

:::success Congratulations!
You have successfully managed your `Filter` subscriptions to handle node disconnections in your application.
:::