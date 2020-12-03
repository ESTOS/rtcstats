## rtcstats.js
Low-level logging on peerconnection API calls and periodic getStats calls for analytics/debugging purposes

## Integration
Just one simple step: include rtcstats.js before any of your webrtc javascript.
```html
<script src='/path/to/rtcstats.js'></script>
```

It will transparently modify the RTCPeerConnection objects and start sending data.
If you need things like a client or conference identifier to be sent along, the recommended way is to use the legacy peerconnection constraints when constructing your RTCPeerConnection like this:

```javascript
var pc = new RTCPeerConnection(yourConfiguration, {
  optional: [
    {rtcStatsClientId: "your client identifier"},
    {rtcStatsPeerId: "identifier for the current peer"},
    {rtcStatsConferenceId: "identifier for the conference, e.g. room name"}
  ]
})
```

The new peerConnection object offers additional functions and callbacks:
```javascript
/**
 * This is the rtcstatistic id ("PC_0", "PC_1", ...) assigned while constructing the peerConnection object
 */
var assignedID = pc.getRtcStatsId();

/**
 * This is an optional callback that will be called after statistics have been fetched from the peerConnection
 * It can be used to
 * * modify or filter data from the statistics 
 * * add own data to the statistics you need to have in the results 
 *   (reflect status changes on the signalling layer you want to see in association with the statistics)
 *   (e.g. switching from simulcast layers)
 */
 pc.statsCallback = function(rawData: RTCStatsReport) => RTCStatsReport


```


If that integration is not possible there is a fallback integration which allows
sending per-client information about the user id and conference id. This
can be used by calling
```
trace('identity', null, {user: 'your client identifier',
    conference:'identifier for the conference, e.g. room name'});
```

### Requiring as module

#### build

in the root directory of the project:

```bash
$npm i
...
$npm run dist
```

this will create the output in `./out/`

#### require

```javascript
const trace = require("rtcstats/trace-ws")("wss://rtcstats.appear.in"); // url-to-your-websocket-server
require("rtcstats")(
   trace,
   1000, // interval at which getStats will be polled.
   ['', 'webkit', 'moz'] // RTCPeerConnection prefixes to wrap.
);
```

When using ontop of adapter it is typically not necessary (and potentially harmful) to shim the webkit and moz prefixes in addition to the unprefixed version.

## Importing the dumps
The dumps generated can be imported and visualized using [this tool](https://fippo.github.io/webrtc-dump-importer/rtcstats)
