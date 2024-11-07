# @simPRO-Software/stripe-terminal

Stripe SDK bindings for Capacitor Applications. __This plugin is still in the RC (release candidate) phase.__
We have confirmed that it works well in the demo project. Please refer to <https://github.com/simPRO-Software/stripe/tree/main/demo/angular> for the implementation.

## Install

```bash
npm install @simPRO-Software/stripe-terminal
npx cap sync
```

### Web

No additional steps are necessary.

__Note: Stripe Web SDK is beta version. So this plugin's implement is experimental. Please refer to <https://github.com/stripe/terminal-js> for more information.__

### iOS

- [iOS Configure your app](https://stripe.com/docs/terminal/payments/setup-integration?terminal-sdk-platform=ios#configure)

### Android

Add permissions to your `android/app/src/main/AndroidManifest.xml` file:

```diff
+ <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
+ <uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
+ <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30" />
+ <uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
+ <uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />
+ <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
```

If used in conjunction with the `@simPRO-Software/stripe` plugin, the following settings may be necessary

Add packagingOptions to your `android/app/build.gradle` file:

```diff
android {
...
+  packagingOptions {
+    resources.excludes.add("org/bouncycastle/x509/*")
+  }
}
```

And update minSdkVersion to 26 And compileSdkVersion to 34 in your `android/app/build.gradle` file:

```diff
  ext {
-    minSdkVersion = 22
-    compileSdkVersion = 33
+    minSdkVersion = 30
+    compileSdkVersion = 34
```

## Usage

### Simple collect payment

#### Use plugin client

```typescript
(async ()=> {
  /**
   * tokenProviderEndpoint: The URL of your backend to provide a token. Use Post request to get a token.
   */
  await StripeTerminal.initialize({ tokenProviderEndpoint: 'https://example.com/token', isTest: true })
  const { readers } = await StripeTerminal.discoverReaders({
    type: TerminalConnectTypes.TapToPay,
    locationId: "**************",
  });
  await StripeTerminal.connectReader({
    reader: readers[0],
  });
  // Collect payment intent
  await StripeTerminal.collectPaymentMethod({ paymentIntent: "**************" });
  // Process and confirm payment intent
  await StripeTerminal.confirmPaymentIntent();
  // disconnect reader
  await StripeTerminal.disconnectReader();
});
```

#### set string token

```typescript
(async ()=> {
  // run before StripeTerminal.initialize
  StripeTerminal.addListener(TerminalEventsEnum.RequestedConnectionToken, async () => {
    const { token } = (await fetch("https://example.com/token")).json();
    StripeTerminal.setConnectionToken({ token });
  });
});
(async ()=> {
  await StripeTerminal.initialize({ isTest: true })
  const { readers } = await StripeTerminal.discoverReaders({
    type: TerminalConnectTypes.TapToPay,
    locationId: "**************",
  });
  await StripeTerminal.connectReader({
    reader: readers[0],
  });
  // Collect payment intent
  await StripeTerminal.collectPaymentMethod({ paymentIntent: "**************" });
  // Process and confirm payment intent
  await StripeTerminal.confirmPaymentIntent();
  // disconnect reader
  await StripeTerminal.disconnectReader();
});
````

### Listen device update

The device will __if necessary__ automatically start updating itself. It is important to handle them as needed so as not to disrupt business operations.

```ts
(async ()=> {
  StripeTerminal.addListener(TerminalEventsEnum.ReportAvailableUpdate, async ({ update }) => {
    if (window.confirm("Will you update the device?")) {
      await StripeTerminal.installAvailableUpdate();
    }
  });
  StripeTerminal.addListener(TerminalEventsEnum.StartInstallingUpdate, async ({ update }) => {
    console.log(update);
    if (window.confirm("Will you interrupt the update?")) {
      StripeTerminal.cancelInstallUpdate();
    }
  });
  StripeTerminal.addListener(TerminalEventsEnum.ReaderSoftwareUpdateProgress, async ({ progress }) => {
    // be able to use this value to create a progress bar.
  });
  StripeTerminal.addListener(TerminalEventsEnum.FinishInstallingUpdate, async ({ update }) => {
    console.log(update);
  });
});
```

### Get terminal processing information

For devices without leader screen, processing information must be retrieved and displayed on the mobile device. Get it with a listener.

```ts
/**
 * Listen battery level. If the battery level is low, you can notify the user to charge the device.
 */
StripeTerminal.addListener(TerminalEventsEnum.BatteryLevel, async ({ level, charging, status }) => {
  console.log(level, charging, status);
});

/**
 * Listen reader event. You can get the reader's status and display it on the mobile device.
 */
StripeTerminal.addListener(TerminalEventsEnum.ReaderEvent, async ({ event }) => {
  console.log(event);
});

/**
 * Listen display message. You can get the message to be displayed on the mobile device.
 */
StripeTerminal.addListener(TerminalEventsEnum.RequestDisplayMessage, async ({ messageType, message }) => {
  console.log(messageType, message);
});

/**
 * Listen reader input. You can get the message what can be used for payment.
 */
StripeTerminal.addListener(TerminalEventsEnum.RequestReaderInput, async ({ options, message }) => {
  console.log(options, message);
});
```

### More details on the leader screen

The contents of the payment can be shown on the display. This requires a leader screen on the device.
This should be run before `collectPaymentMethod`.

```ts
await StripeTerminal.setReaderDisplay({
  currency: 'usd',
  tax: 0,
  total: 1000,
  lineItems: [{
    displayName: 'winecode',
    quantity: 2,
    amount: 500
  }] as CartLineItem[],
})

// Of course, erasure is also possible.
await StripeTerminal.clearReaderDisplay();
```

### Simulate reader status changes for testing

To implement updates, etc., we are facilitating an API to change the state of the simulator. This should be done before discoverReaders.

```ts
await StripeTerminal.setSimulatorConfiguration({ update: SimulateReaderUpdate.UpdateAvailable })
```

## API

<docgen-index>

- [`initialize(...)`](#initialize)
- [`discoverReaders(...)`](#discoverreaders)
- [`setConnectionToken(...)`](#setconnectiontoken)
- [`setSimulatorConfiguration(...)`](#setsimulatorconfiguration)
- [`connectReader(...)`](#connectreader)
- [`getConnectedReader()`](#getconnectedreader)
- [`disconnectReader()`](#disconnectreader)
- [`cancelDiscoverReaders()`](#canceldiscoverreaders)
- [`collectPaymentMethod(...)`](#collectpaymentmethod)
- [`cancelCollectPaymentMethod()`](#cancelcollectpaymentmethod)
- [`confirmPaymentIntent()`](#confirmpaymentintent)
- [`installAvailableUpdate()`](#installavailableupdate)
- [`cancelInstallUpdate()`](#cancelinstallupdate)
- [`setReaderDisplay(...)`](#setreaderdisplay)
- [`clearReaderDisplay()`](#clearreaderdisplay)
- [`rebootReader()`](#rebootreader)
- [`cancelReaderReconnection()`](#cancelreaderreconnection)
- [`addListener(TerminalEventsEnum.Loaded, ...)`](#addlistenerterminaleventsenumloaded)
- [`addListener(TerminalEventsEnum.RequestedConnectionToken, ...)`](#addlistenerterminaleventsenumrequestedconnectiontoken)
- [`addListener(TerminalEventsEnum.DiscoveredReaders, ...)`](#addlistenerterminaleventsenumdiscoveredreaders)
- [`addListener(TerminalEventsEnum.ConnectedReader, ...)`](#addlistenerterminaleventsenumconnectedreader)
- [`addListener(TerminalEventsEnum.DisconnectedReader, ...)`](#addlistenerterminaleventsenumdisconnectedreader)
- [`addListener(TerminalEventsEnum.ConnectionStatusChange, ...)`](#addlistenerterminaleventsenumconnectionstatuschange)
- [`addListener(TerminalEventsEnum.UnexpectedReaderDisconnect, ...)`](#addlistenerterminaleventsenumunexpectedreaderdisconnect)
- [`addListener(TerminalEventsEnum.ConfirmedPaymentIntent, ...)`](#addlistenerterminaleventsenumconfirmedpaymentintent)
- [`addListener(TerminalEventsEnum.CollectedPaymentIntent, ...)`](#addlistenerterminaleventsenumcollectedpaymentintent)
- [`addListener(TerminalEventsEnum.Canceled, ...)`](#addlistenerterminaleventsenumcanceled)
- [`addListener(TerminalEventsEnum.Failed, ...)`](#addlistenerterminaleventsenumfailed)
- [`addListener(TerminalEventsEnum.ReportAvailableUpdate, ...)`](#addlistenerterminaleventsenumreportavailableupdate)
- [`addListener(TerminalEventsEnum.StartInstallingUpdate, ...)`](#addlistenerterminaleventsenumstartinstallingupdate)
- [`addListener(TerminalEventsEnum.ReaderSoftwareUpdateProgress, ...)`](#addlistenerterminaleventsenumreadersoftwareupdateprogress)
- [`addListener(TerminalEventsEnum.FinishInstallingUpdate, ...)`](#addlistenerterminaleventsenumfinishinstallingupdate)
- [`addListener(TerminalEventsEnum.BatteryLevel, ...)`](#addlistenerterminaleventsenumbatterylevel)
- [`addListener(TerminalEventsEnum.ReaderEvent, ...)`](#addlistenerterminaleventsenumreaderevent)
- [`addListener(TerminalEventsEnum.RequestDisplayMessage, ...)`](#addlistenerterminaleventsenumrequestdisplaymessage)
- [`addListener(TerminalEventsEnum.RequestReaderInput, ...)`](#addlistenerterminaleventsenumrequestreaderinput)
- [`addListener(TerminalEventsEnum.PaymentStatusChange, ...)`](#addlistenerterminaleventsenumpaymentstatuschange)
- [`addListener(TerminalEventsEnum.ReaderReconnectStarted, ...)`](#addlistenerterminaleventsenumreaderreconnectstarted)
- [`addListener(TerminalEventsEnum.ReaderReconnectSucceeded, ...)`](#addlistenerterminaleventsenumreaderreconnectsucceeded)
- [`addListener(TerminalEventsEnum.ReaderReconnectFailed, ...)`](#addlistenerterminaleventsenumreaderreconnectfailed)
- [Interfaces](#interfaces)
- [Type Aliases](#type-aliases)
- [Enums](#enums)

</docgen-index>

<docgen-api>
<!--Update the source file JSDoc comments and rerun docgen to update the docs below-->

### initialize(...)

```typescript
initialize(options: { tokenProviderEndpoint?: string; isTest: boolean; }) => Promise<void>
```

| Param         | Type                                                              |
| ------------- | ----------------------------------------------------------------- |
| __`options`__ | <code>{ tokenProviderEndpoint?: string; isTest: boolean; }</code> |

--------------------

### discoverReaders(...)

```typescript
discoverReaders(options: { type: TerminalConnectTypes; locationId?: string; }) => Promise<{ readers: ReaderInterface[]; }>
```

| Param         | Type                                                                                                  |
| ------------- | ----------------------------------------------------------------------------------------------------- |
| __`options`__ | <code>{ type: <a href="#terminalconnecttypes">TerminalConnectTypes</a>; locationId?: string; }</code> |

__Returns:__ <code>Promise&lt;{ readers: ReaderInterface[]; }&gt;</code>

--------------------

### setConnectionToken(...)

```typescript
setConnectionToken(options: { token: string; }) => Promise<void>
```

| Param         | Type                            |
| ------------- | ------------------------------- |
| __`options`__ | <code>{ token: string; }</code> |

--------------------

### setSimulatorConfiguration(...)

```typescript
setSimulatorConfiguration(options: { update?: SimulateReaderUpdate; simulatedCard?: SimulatedCardType; simulatedTipAmount?: number; }) => Promise<void>
```

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.models/-simulator-configuration/index.html)

| Param         | Type                                                                                                                                                                                 |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| __`options`__ | <code>{ update?: <a href="#simulatereaderupdate">SimulateReaderUpdate</a>; simulatedCard?: <a href="#simulatedcardtype">SimulatedCardType</a>; simulatedTipAmount?: number; }</code> |

--------------------

### connectReader(...)

```typescript
connectReader(options: { reader: ReaderInterface; autoReconnectOnUnexpectedDisconnect?: boolean; merchantDisplayName?: string; onBehalfOf?: string; }) => Promise<void>
```

| Param         | Type                                                                                                                                                                       |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| __`options`__ | <code>{ reader: <a href="#readerinterface">ReaderInterface</a>; autoReconnectOnUnexpectedDisconnect?: boolean; merchantDisplayName?: string; onBehalfOf?: string; }</code> |

--------------------

### getConnectedReader()

```typescript
getConnectedReader() => Promise<{ reader: ReaderInterface | null; }>
```

__Returns:__ <code>Promise&lt;{ reader: <a href="#readerinterface">ReaderInterface</a> | null; }&gt;</code>

--------------------

### disconnectReader()

```typescript
disconnectReader() => Promise<void>
```

--------------------

### cancelDiscoverReaders()

```typescript
cancelDiscoverReaders() => Promise<void>
```

--------------------

### collectPaymentMethod(...)

```typescript
collectPaymentMethod(options: { paymentIntent: string; }) => Promise<void>
```

| Param         | Type                                    |
| ------------- | --------------------------------------- |
| __`options`__ | <code>{ paymentIntent: string; }</code> |

--------------------

### cancelCollectPaymentMethod()

```typescript
cancelCollectPaymentMethod() => Promise<void>
```

--------------------

### confirmPaymentIntent()

```typescript
confirmPaymentIntent() => Promise<void>
```

--------------------

### installAvailableUpdate()

```typescript
installAvailableUpdate() => Promise<void>
```

--------------------

### cancelInstallUpdate()

```typescript
cancelInstallUpdate() => Promise<void>
```

--------------------

### setReaderDisplay(...)

```typescript
setReaderDisplay(options: Cart) => Promise<void>
```

| Param         | Type                                  |
| ------------- | ------------------------------------- |
| __`options`__ | <code><a href="#cart">Cart</a></code> |

--------------------

### clearReaderDisplay()

```typescript
clearReaderDisplay() => Promise<void>
```

--------------------

### rebootReader()

```typescript
rebootReader() => Promise<void>
```

--------------------

### cancelReaderReconnection()

```typescript
cancelReaderReconnection() => Promise<void>
```

--------------------

### addListener(TerminalEventsEnum.Loaded, ...)

```typescript
addListener(eventName: TerminalEventsEnum.Loaded, listenerFunc: () => void) => Promise<PluginListenerHandle>
```

| Param              | Type                                                                     |
| ------------------ | ------------------------------------------------------------------------ |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.Loaded</a></code> |
| __`listenerFunc`__ | <code>() =&gt; void</code>                                               |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.RequestedConnectionToken, ...)

```typescript
addListener(eventName: TerminalEventsEnum.RequestedConnectionToken, listenerFunc: () => void) => Promise<PluginListenerHandle>
```

| Param              | Type                                                                                       |
| ------------------ | ------------------------------------------------------------------------------------------ |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.RequestedConnectionToken</a></code> |
| __`listenerFunc`__ | <code>() =&gt; void</code>                                                                 |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.DiscoveredReaders, ...)

```typescript
addListener(eventName: TerminalEventsEnum.DiscoveredReaders, listenerFunc: ({ readers }: { readers: ReaderInterface[]; }) => void) => Promise<PluginListenerHandle>
```

| Param              | Type                                                                                |
| ------------------ | ----------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.DiscoveredReaders</a></code> |
| __`listenerFunc`__ | <code>({ readers }: { readers: ReaderInterface[]; }) =&gt; void</code>              |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.ConnectedReader, ...)

```typescript
addListener(eventName: TerminalEventsEnum.ConnectedReader, listenerFunc: () => void) => Promise<PluginListenerHandle>
```

| Param              | Type                                                                              |
| ------------------ | --------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.ConnectedReader</a></code> |
| __`listenerFunc`__ | <code>() =&gt; void</code>                                                        |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.DisconnectedReader, ...)

```typescript
addListener(eventName: TerminalEventsEnum.DisconnectedReader, listenerFunc: ({ reason }: { reason?: DisconnectReason | undefined; }) => void) => Promise<PluginListenerHandle>
```

Emitted when the reader is disconnected, either in response to [`disconnectReader()`](#disconnectreader)
or some connection error.

For all reader types, this is emitted in response to [`disconnectReader()`](#disconnectreader)
without a `reason` property.

For Bluetooth and USB readers, this is emitted with a `reason` property when the reader disconnects.

__Note:__ For Bluetooth and USB readers, when you call [`disconnectReader()`](#disconnectreader), this event
will be emitted twice: one without a `reason` in acknowledgement of your call, and again with a `reason` when the reader
finishes disconnecting.

| Param              | Type                                                                                                 |
| ------------------ | ---------------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.DisconnectedReader</a></code>                 |
| __`listenerFunc`__ | <code>({ reason }: { reason?: <a href="#disconnectreason">DisconnectReason</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.ConnectionStatusChange, ...)

```typescript
addListener(eventName: TerminalEventsEnum.ConnectionStatusChange, listenerFunc: ({ status }: { status: ConnectionStatus; }) => void) => Promise<PluginListenerHandle>
```

Emitted when the Terminal's connection status changed.

Note: You should *not* use this method to detect when a reader unexpectedly disconnects from your app,
as it cannot be used to accurately distinguish between expected and unexpected disconnect events.

To detect unexpected disconnects (e.g. to automatically notify your user), you should instead use
the UnexpectedReaderDisconnect event.

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-terminal-listener/on-connection-status-change.html)

| Param              | Type                                                                                                |
| ------------------ | --------------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.ConnectionStatusChange</a></code>            |
| __`listenerFunc`__ | <code>({ status }: { status: <a href="#connectionstatus">ConnectionStatus</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.UnexpectedReaderDisconnect, ...)

```typescript
addListener(eventName: TerminalEventsEnum.UnexpectedReaderDisconnect, listenerFunc: ({ reader }: { reader: ReaderInterface; }) => void) => Promise<PluginListenerHandle>
```

The Terminal disconnected unexpectedly from the reader.

In your implementation of this method, you may want to notify your user that the reader disconnected.
You may also call [`discoverReaders()`](#discoverreaders) to begin scanning for readers, and attempt
to automatically reconnect to the disconnected reader. Be sure to either set a timeout or make it
possible to cancel calls to `discoverReaders()`

When connected to a Bluetooth or USB reader, you can get more information about the disconnect by
implementing the DisconnectedReader event.

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-terminal-listener/on-unexpected-reader-disconnect.html)

| Param              | Type                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.UnexpectedReaderDisconnect</a></code>      |
| __`listenerFunc`__ | <code>({ reader }: { reader: <a href="#readerinterface">ReaderInterface</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.ConfirmedPaymentIntent, ...)

```typescript
addListener(eventName: TerminalEventsEnum.ConfirmedPaymentIntent, listenerFunc: () => void) => Promise<PluginListenerHandle>
```

| Param              | Type                                                                                     |
| ------------------ | ---------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.ConfirmedPaymentIntent</a></code> |
| __`listenerFunc`__ | <code>() =&gt; void</code>                                                               |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.CollectedPaymentIntent, ...)

```typescript
addListener(eventName: TerminalEventsEnum.CollectedPaymentIntent, listenerFunc: () => void) => Promise<PluginListenerHandle>
```

| Param              | Type                                                                                     |
| ------------------ | ---------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.CollectedPaymentIntent</a></code> |
| __`listenerFunc`__ | <code>() =&gt; void</code>                                                               |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.Canceled, ...)

```typescript
addListener(eventName: TerminalEventsEnum.Canceled, listenerFunc: () => void) => Promise<PluginListenerHandle>
```

Emitted when [`cancelCollectPaymentMethod()`](#cancelcollectpaymentmethod) is called and succeeds.
The Promise returned by `cancelCollectPaymentMethod()` will also be resolved.

| Param              | Type                                                                       |
| ------------------ | -------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.Canceled</a></code> |
| __`listenerFunc`__ | <code>() =&gt; void</code>                                                 |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.Failed, ...)

```typescript
addListener(eventName: TerminalEventsEnum.Failed, listenerFunc: (info: { message: string; code?: string; declineCode?: string; }) => void) => Promise<PluginListenerHandle>
```

Emitted when either [`collectPaymentMethod()`](#collectpaymentmethod) or [`confirmPaymentIntent()`](#confirmpaymentintent)
fails. The Promise returned by the relevant call will also be rejected.

| Param              | Type                                                                                      |
| ------------------ | ----------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.Failed</a></code>                  |
| __`listenerFunc`__ | <code>(info: { message: string; code?: string; declineCode?: string; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.ReportAvailableUpdate, ...)

```typescript
addListener(eventName: TerminalEventsEnum.ReportAvailableUpdate, listenerFunc: ({ update, }: { update: ReaderSoftwareUpdateInterface; }) => void) => Promise<PluginListenerHandle>
```

Emitted when a software update is available for the connected reader.

| Param              | Type                                                                                                                           |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.ReportAvailableUpdate</a></code>                                        |
| __`listenerFunc`__ | <code>({ update, }: { update: <a href="#readersoftwareupdateinterface">ReaderSoftwareUpdateInterface</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.StartInstallingUpdate, ...)

```typescript
addListener(eventName: TerminalEventsEnum.StartInstallingUpdate, listenerFunc: ({ update, }: { update: ReaderSoftwareUpdateInterface; }) => void) => Promise<PluginListenerHandle>
```

__Only applicable to Bluetooth and USB readers.__

Emitted when the connected reader begins installing a software update.
If a mandatory software update is available when a reader first connects, that update is
automatically installed. The update will be installed before ConnectedReader is emitted and
before the Promise returned by [`connectReader()`](#connectreader) resolves.
In this case, you will receive this sequence of events:

1. StartInstallingUpdate
2. ReaderSoftwareUpdateProgress (repeatedly)
3. FinishInstallingUpdates
4. ConnectedReader
5. `connectReader()` Promise resolves

Your app should show UI to the user indiciating that a software update is being installed
to explain why connecting is taking longer than usual.

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-reader-listener/on-start-installing-update.html)

| Param              | Type                                                                                                                           |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.StartInstallingUpdate</a></code>                                        |
| __`listenerFunc`__ | <code>({ update, }: { update: <a href="#readersoftwareupdateinterface">ReaderSoftwareUpdateInterface</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.ReaderSoftwareUpdateProgress, ...)

```typescript
addListener(eventName: TerminalEventsEnum.ReaderSoftwareUpdateProgress, listenerFunc: ({ progress }: { progress: number; }) => void) => Promise<PluginListenerHandle>
```

__Only applicable to Bluetooth and USB readers.__

Emitted periodically while reader software is updating to inform of the installation progress.
`progress` is a float between 0 and 1.

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-reader-listener/on-report-reader-software-update-progress.html)

| Param              | Type                                                                                           |
| ------------------ | ---------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.ReaderSoftwareUpdateProgress</a></code> |
| __`listenerFunc`__ | <code>({ progress }: { progress: number; }) =&gt; void</code>                                  |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.FinishInstallingUpdate, ...)

```typescript
addListener(eventName: TerminalEventsEnum.FinishInstallingUpdate, listenerFunc: (args: { update: ReaderSoftwareUpdateInterface; } | { error: string; }) => void) => Promise<PluginListenerHandle>
```

__Only applicable to Bluetooth and USB readers.__

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-reader-listener/on-finish-installing-update.html)

| Param              | Type                                                                                                                                          |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.FinishInstallingUpdate</a></code>                                                      |
| __`listenerFunc`__ | <code>(args: { update: <a href="#readersoftwareupdateinterface">ReaderSoftwareUpdateInterface</a>; } \| { error: string; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.BatteryLevel, ...)

```typescript
addListener(eventName: TerminalEventsEnum.BatteryLevel, listenerFunc: ({ level, charging, status, }: { level: number; charging: boolean; status: BatteryStatus; }) => void) => Promise<PluginListenerHandle>
```

__Only applicable to Bluetooth and USB readers.__

Emitted upon connection and every 10 minutes.

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-reader-listener/on-battery-level-update.html)

| Param              | Type                                                                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.BatteryLevel</a></code>                                                                    |
| __`listenerFunc`__ | <code>({ level, charging, status, }: { level: number; charging: boolean; status: <a href="#batterystatus">BatteryStatus</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.ReaderEvent, ...)

```typescript
addListener(eventName: TerminalEventsEnum.ReaderEvent, listenerFunc: ({ event }: { event: ReaderEvent; }) => void) => Promise<PluginListenerHandle>
```

__Only applicable to Bluetooth and USB readers.__

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-reader-listenable/on-report-reader-event.html)

| Param              | Type                                                                                    |
| ------------------ | --------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.ReaderEvent</a></code>           |
| __`listenerFunc`__ | <code>({ event }: { event: <a href="#readerevent">ReaderEvent</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.RequestDisplayMessage, ...)

```typescript
addListener(eventName: TerminalEventsEnum.RequestDisplayMessage, listenerFunc: ({ messageType, message, }: { messageType: ReaderDisplayMessage; message: string; }) => void) => Promise<PluginListenerHandle>
```

__Only applicable to Bluetooth and USB readers.__

Emitted when the Terminal requests that a message be displayed in your app.

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-reader-listener/on-request-reader-display-message.html)

| Param              | Type                                                                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.RequestDisplayMessage</a></code>                                                          |
| __`listenerFunc`__ | <code>({ messageType, message, }: { messageType: <a href="#readerdisplaymessage">ReaderDisplayMessage</a>; message: string; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.RequestReaderInput, ...)

```typescript
addListener(eventName: TerminalEventsEnum.RequestReaderInput, listenerFunc: ({ options, message, }: { options: ReaderInputOption[]; message: string; }) => void) => Promise<PluginListenerHandle>
```

__Only applicable to Bluetooth and USB readers.__

Emitted when the reader begins waiting for input. Your app should prompt the customer
to present a source using one of the given input options. If the reader emits a message,
the RequestDisplayMessage event will be emitted.

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-reader-listener/on-request-reader-input.html)

| Param              | Type                                                                                                |
| ------------------ | --------------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.RequestReaderInput</a></code>                |
| __`listenerFunc`__ | <code>({ options, message, }: { options: ReaderInputOption[]; message: string; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.PaymentStatusChange, ...)

```typescript
addListener(eventName: TerminalEventsEnum.PaymentStatusChange, listenerFunc: ({ status }: { status: PaymentStatus; }) => void) => Promise<PluginListenerHandle>
```

[*Stripe docs reference*](https://stripe.dev/stripe-terminal-android/external/com.stripe.stripeterminal.external.callable/-terminal-listener/on-payment-status-change.html)

| Param              | Type                                                                                          |
| ------------------ | --------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.PaymentStatusChange</a></code>         |
| __`listenerFunc`__ | <code>({ status }: { status: <a href="#paymentstatus">PaymentStatus</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.ReaderReconnectStarted, ...)

```typescript
addListener(eventName: TerminalEventsEnum.ReaderReconnectStarted, listenerFunc: ({ reader, reason, }: { reader: ReaderInterface; reason: string; }) => void) => Promise<PluginListenerHandle>
```

| Param              | Type                                                                                                                       |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.ReaderReconnectStarted</a></code>                                   |
| __`listenerFunc`__ | <code>({ reader, reason, }: { reader: <a href="#readerinterface">ReaderInterface</a>; reason: string; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.ReaderReconnectSucceeded, ...)

```typescript
addListener(eventName: TerminalEventsEnum.ReaderReconnectSucceeded, listenerFunc: ({ reader }: { reader: ReaderInterface; }) => void) => Promise<PluginListenerHandle>
```

| Param              | Type                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.ReaderReconnectSucceeded</a></code>        |
| __`listenerFunc`__ | <code>({ reader }: { reader: <a href="#readerinterface">ReaderInterface</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### addListener(TerminalEventsEnum.ReaderReconnectFailed, ...)

```typescript
addListener(eventName: TerminalEventsEnum.ReaderReconnectFailed, listenerFunc: ({ reader }: { reader: ReaderInterface; }) => void) => Promise<PluginListenerHandle>
```

| Param              | Type                                                                                              |
| ------------------ | ------------------------------------------------------------------------------------------------- |
| __`eventName`__    | <code><a href="#terminaleventsenum">TerminalEventsEnum.ReaderReconnectFailed</a></code>           |
| __`listenerFunc`__ | <code>({ reader }: { reader: <a href="#readerinterface">ReaderInterface</a>; }) =&gt; void</code> |

__Returns:__ <code>Promise&lt;<a href="#pluginlistenerhandle">PluginListenerHandle</a>&gt;</code>

--------------------

### Interfaces

#### PluginListenerHandle

| Prop         | Type                                      |
| ------------ | ----------------------------------------- |
| __`remove`__ | <code>() =&gt; Promise&lt;void&gt;</code> |

### Type Aliases

#### ReaderInterface

<code>{ /** *The unique serial number is primary identifier inner plugin. */ serialNumber: string; label: string; batteryLevel: number; batteryStatus: <a href="#batterystatus">BatteryStatus</a>; simulated: boolean; id: number; availableUpdate: <a href="#readersoftwareupdateinterface">ReaderSoftwareUpdateInterface</a>; locationId: string; ipAddress: string; status: <a href="#networkstatus">NetworkStatus</a>; location: <a href="#locationinterface">LocationInterface</a>; locationStatus: <a href="#locationstatus">LocationStatus</a>; deviceType: <a href="#devicetype">DeviceType</a>; deviceSoftwareVersion: string | null; /** *iOS Only properties. These properties are not available on Android. */ isCharging: number; /** *Android Only properties. These properties are not available on iOS. */ baseUrl: string; bootloaderVersion: string; configVersion: string; emvKeyProfileId: string; firmwareVersion: string; hardwareVersion: string; macKeyProfileId: string; pinKeyProfileId: string; trackKeyProfileId: string; settingsVersion: string; pinKeysetId: string; /** *@deprecated This property has been deprecated and should use the `serialNumber` property.*/ index?: number; }</code>

#### ReaderSoftwareUpdateInterface

<code>{ deviceSoftwareVersion: string; estimatedUpdateTime: <a href="#updatetimeestimate">UpdateTimeEstimate</a>; requiredAt: number; }</code>

#### LocationInterface

<code>{ id: string; displayName: string; address: { city: string; country: string; postalCode: string; line1: string; line2: string; state: string; }; ipAddress: string; }</code>

#### DeviceType

<code>Stripe.Terminal.Reader.<a href="#devicetype">DeviceType</a></code>

#### Cart

<code>{ currency: string; tax: number; total: number; lineItems: CartLineItem[]; }</code>

#### CartLineItem

<code>{ displayName: string; quantity: number; amount: number; }</code>

### Enums

#### BatteryStatus

| Members        | Value                   |
| -------------- | ----------------------- |
| __`Unknown`__  | <code>'UNKNOWN'</code>  |
| __`Critical`__ | <code>'CRITICAL'</code> |
| __`Low`__      | <code>'LOW'</code>      |
| __`Nominal`__  | <code>'NOMINAL'</code>  |

#### UpdateTimeEstimate

| Members                    | Value                                  |
| -------------------------- | -------------------------------------- |
| __`LessThanOneMinute`__    | <code>'LESS_THAN_ONE_MINUTE'</code>    |
| __`OneToTwoMinutes`__      | <code>'ONE_TO_TWO_MINUTES'</code>      |
| __`TwoToFiveMinutes`__     | <code>'TWO_TO_FIVE_MINUTES'</code>     |
| __`FiveToFifteenMinutes`__ | <code>'FIVE_TO_FIFTEEN_MINUTES'</code> |

#### NetworkStatus

| Members       | Value                  |
| ------------- | ---------------------- |
| __`Unknown`__ | <code>'UNKNOWN'</code> |
| __`Online`__  | <code>'ONLINE'</code>  |
| __`Offline`__ | <code>'OFFLINE'</code> |

#### LocationStatus

| Members       | Value                  |
| ------------- | ---------------------- |
| __`NotSet`__  | <code>'NOT_SET'</code> |
| __`Set`__     | <code>'SET'</code>     |
| __`Unknown`__ | <code>'UNKNOWN'</code> |

#### DeviceType

| Members                | Value                           |
| ---------------------- | ------------------------------- |
| __`cotsDevice`__       | <code>'cotsDevice'</code>       |
| __`wisePad3s`__        | <code>'wisePad3s'</code>        |
| __`appleBuiltIn`__     | <code>'appleBuiltIn'</code>     |
| __`chipper1X`__        | <code>'chipper1X'</code>        |
| __`chipper2X`__        | <code>'chipper2X'</code>        |
| __`etna`__             | <code>'etna'</code>             |
| __`stripeM2`__         | <code>'stripeM2'</code>         |
| __`stripeS700`__       | <code>'stripeS700'</code>       |
| __`stripeS700DevKit`__ | <code>'stripeS700Devkit'</code> |
| __`verifoneP400`__     | <code>'verifoneP400'</code>     |
| __`wiseCube`__         | <code>'wiseCube'</code>         |
| __`wisePad3`__         | <code>'wisePad3'</code>         |
| __`wisePosE`__         | <code>'wisePosE'</code>         |
| __`wisePosEDevKit`__   | <code>'wisePosEDevkit'</code>   |
| __`unknown`__          | <code>'unknown'</code>          |

#### TerminalConnectTypes

| Members         | Value                     |
| --------------- | ------------------------- |
| __`Simulated`__ | <code>'simulated'</code>  |
| __`Internet`__  | <code>'internet'</code>   |
| __`Bluetooth`__ | <code>'bluetooth'</code>  |
| __`Usb`__       | <code>'usb'</code>        |
| __`TapToPay`__  | <code>'tap-to-pay'</code> |

#### SimulateReaderUpdate

| Members                        | Value                                      |
| ------------------------------ | ------------------------------------------ |
| __`UpdateAvailable`__          | <code>'UPDATE_AVAILABLE'</code>            |
| __`None`__                     | <code>'NONE'</code>                        |
| __`Required`__                 | <code>'REQUIRED'</code>                    |
| __`Random`__                   | <code>'RANDOM'</code>                      |
| __`LowBattery`__               | <code>'LOW_BATTERY'</code>                 |
<!-- | **`LowBatterySucceedConnect`** | <code>'LOW_BATTERY_SUCCEED_CONNECT'</code> | -->

#### SimulatedCardType

| Members                               | Value                                             |
| ------------------------------------- | ------------------------------------------------- |
| __`Visa`__                            | <code>'VISA'</code>                               |
| __`VisaDebit`__                       | <code>'VISA_DEBIT'</code>                         |
| __`Mastercard`__                      | <code>'MASTERCARD'</code>                         |
| __`MastercardDebit`__                 | <code>'MASTERCARD_DEBIT'</code>                   |
| __`MastercardPrepaid`__               | <code>'MASTERCARD_PREPAID'</code>                 |
| __`Amex`__                            | <code>'AMEX'</code>                               |
| __`Amex2`__                           | <code>'AMEX_2'</code>                             |
| __`Discover`__                        | <code>'DISCOVER'</code>                           |
| __`Discover2`__                       | <code>'DISCOVER_2'</code>                         |
| __`DinersClub`__                      | <code>'DINERS'</code>                             |
| __`DinersClulb14Digits`__             | <code>'DINERS_14_DIGITS'</code>                   |
| __`JCB`__                             | <code>'JCB'</code>                                |
| __`UnionPay`__                        | <code>'UNION_PAY'</code>                          |
| __`Interac`__                         | <code>'INTERAC'</code>                            |
| __`EftposAustraliaDebit`__            | <code>'EFTPOS_AU_DEBIT'</code>                    |
| __`VisaUsCommonDebit`__               | <code>'VISA_US_COMMON_DEBIT'</code>               |
| __`ChargeDeclined`__                  | <code>'CHARGE_DECLINED'</code>                    |
| __`ChargeDeclinedInsufficientFunds`__ | <code>'CHARGE_DECLINED_INSUFFICIENT_FUNDS'</code> |
| __`ChargeDeclinedLostCard`__          | <code>'CHARGE_DECLINED_LOST_CARD'</code>          |
| __`ChargeDeclinedStolenCard`__        | <code>'CHARGE_DECLINED_STOLEN_CARD'</code>        |
| __`ChargeDeclinedExpiredCard`__       | <code>'CHARGE_DECLINED_EXPIRED_CARD'</code>       |
| __`ChargeDeclinedProcessingError`__   | <code>'CHARGE_DECLINED_PROCESSING_ERROR'</code>   |
| __`EftposAustraliaVisaDebit`__        | <code>'EFTPOS_AU_VISA_DEBIT'</code>               |
| __`EftposAustraliaMastercardDebit`__  | <code>'EFTPOS_AU_DEBIT_MASTERCARD'</code>         |
| __`OfflinePinCVM`__                   | <code>'OFFLINE_PIN_CVM'</code>                    |
| __`OfflinePinSCARetry`__              | <code>'OFFLINE_PIN_SCA_RETRY'</code>              |
| __`OnlinePinCVM`__                    | <code>'ONLINE_PIN_CVM'</code>                     |
| __`OnlinePinSCARetry`__               | <code>'ONLINE_PIN_SCA_RETRY'</code>               |

#### TerminalEventsEnum

| Members                            | Value                                               |
| ---------------------------------- | --------------------------------------------------- |
| __`Loaded`__                       | <code>'terminalLoaded'</code>                       |
| __`DiscoveredReaders`__            | <code>'terminalDiscoveredReaders'</code>            |
| __`CancelDiscoveredReaders`__      | <code>'terminalCancelDiscoveredReaders'</code>      |
| __`ConnectedReader`__              | <code>'terminalConnectedReader'</code>              |
| __`DisconnectedReader`__           | <code>'terminalDisconnectedReader'</code>           |
| __`ConnectionStatusChange`__       | <code>'terminalConnectionStatusChange'</code>       |
| __`UnexpectedReaderDisconnect`__   | <code>'terminalUnexpectedReaderDisconnect'</code>   |
| __`ConfirmedPaymentIntent`__       | <code>'terminalConfirmedPaymentIntent'</code>       |
| __`CollectedPaymentIntent`__       | <code>'terminalCollectedPaymentIntent'</code>       |
| __`Canceled`__                     | <code>'terminalCanceled'</code>                     |
| __`Failed`__                       | <code>'terminalFailed'</code>                       |
| __`RequestedConnectionToken`__     | <code>'terminalRequestedConnectionToken'</code>     |
| __`ReportAvailableUpdate`__        | <code>'terminalReportAvailableUpdate'</code>        |
| __`StartInstallingUpdate`__        | <code>'terminalStartInstallingUpdate'</code>        |
| __`ReaderSoftwareUpdateProgress`__ | <code>'terminalReaderSoftwareUpdateProgress'</code> |
| __`FinishInstallingUpdate`__       | <code>'terminalFinishInstallingUpdate'</code>       |
| __`BatteryLevel`__                 | <code>'terminalBatteryLevel'</code>                 |
| __`ReaderEvent`__                  | <code>'terminalReaderEvent'</code>                  |
| __`RequestDisplayMessage`__        | <code>'terminalRequestDisplayMessage'</code>        |
| __`RequestReaderInput`__           | <code>'terminalRequestReaderInput'</code>           |
| __`PaymentStatusChange`__          | <code>'terminalPaymentStatusChange'</code>          |
| __`ReaderReconnectStarted`__       | <code>'terminalReaderReconnectStarted'</code>       |
| __`ReaderReconnectSucceeded`__     | <code>'terminalReaderReconnectSucceeded'</code>     |
| __`ReaderReconnectFailed`__        | <code>'terminalReaderReconnectFailed'</code>        |

#### DisconnectReason

| Members                    | Value                                 |
| -------------------------- | ------------------------------------- |
| __`Unknown`__              | <code>'UNKNOWN'</code>                |
| __`DisconnectRequested`__  | <code>'DISCONNECT_REQUESTED'</code>   |
| __`RebootRequested`__      | <code>'REBOOT_REQUESTED'</code>       |
| __`SecurityReboot`__       | <code>'SECURITY_REBOOT'</code>        |
| __`CriticallyLowBattery`__ | <code>'CRITICALLY_LOW_BATTERY'</code> |
| __`PoweredOff`__           | <code>'POWERED_OFF'</code>            |
| __`BluetoothDisabled`__    | <code>'BLUETOOTH_DISABLED'</code>     |

#### ConnectionStatus

| Members            | Value                        |
| ------------------ | ---------------------------- |
| __`Unknown`__      | <code>'UNKNOWN'</code>       |
| __`NotConnected`__ | <code>'NOT_CONNECTED'</code> |
| __`Connecting`__   | <code>'CONNECTING'</code>    |
| __`Connected`__    | <code>'CONNECTED'</code>     |

#### ReaderEvent

| Members            | Value                        |
| ------------------ | ---------------------------- |
| __`Unknown`__      | <code>'UNKNOWN'</code>       |
| __`CardInserted`__ | <code>'CARD_INSERTED'</code> |
| __`CardRemoved`__  | <code>'CARD_REMOVED'</code>  |

#### ReaderDisplayMessage

| Members                                | Value                                              |
| -------------------------------------- | -------------------------------------------------- |
| __`CheckMobileDevice`__                | <code>'CHECK_MOBILE_DEVICE'</code>                 |
| __`RetryCard`__                        | <code>'RETRY_CARD'</code>                          |
| __`InsertCard`__                       | <code>'INSERT_CARD'</code>                         |
| __`InsertOrSwipeCard`__                | <code>'INSERT_OR_SWIPE_CARD'</code>                |
| __`SwipeCard`__                        | <code>'SWIPE_CARD'</code>                          |
| __`RemoveCard`__                       | <code>'REMOVE_CARD'</code>                         |
| __`MultipleContactlessCardsDetected`__ | <code>'MULTIPLE_CONTACTLESS_CARDS_DETECTED'</code> |
| __`TryAnotherReadMethod`__             | <code>'TRY_ANOTHER_READ_METHOD'</code>             |
| __`TryAnotherCard`__                   | <code>'TRY_ANOTHER_CARD'</code>                    |
| __`CardRemovedTooEarly`__              | <code>'CARD_REMOVED_TOO_EARLY'</code>              |

#### ReaderInputOption

| Members           | Value                       |
| ----------------- | --------------------------- |
| __`None`__        | <code>'NONE'</code>         |
| __`Insert`__      | <code>'INSERT'</code>       |
| __`Swipe`__       | <code>'SWIPE'</code>        |
| __`Tap`__         | <code>'TAP'</code>          |
| __`ManualEntry`__ | <code>'MANUAL_ENTRY'</code> |

#### PaymentStatus

| Members               | Value                            |
| --------------------- | -------------------------------- |
| __`Unknown`__         | <code>'UNKNOWN'</code>           |
| __`NotReady`__        | <code>'NOT_READY'</code>         |
| __`Ready`__           | <code>'READY'</code>             |
| __`WaitingForInput`__ | <code>'WAITING_FOR_INPUT'</code> |
| __`Processing`__      | <code>'PROCESSING'</code>        |

</docgen-api>
