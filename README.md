# Bitmovin Player Conviva Analytics Integration

## Compatibility
**This version of the Conviva Analytics Integration works only with Player Version >= 8.2.x.
The recommended and tested version of the Conviva SDK is 2.151.0.36981.** See [CHANGELOG](CHANGELOG.md) for details.

## Getting Started

1. Clone Git repository
2. Install node.js
3. Install required npm packages: [`npm install`](https://www.npmjs.com/)
4. Run tasks:
  * `npm run lint` to lint TypeScript files
  * `npm run build` to build project into `dist` directory
  * `npm run start` to open test page in browser, build and reload changed files automatically

## Usage

1. Build the script by running `npm run build`

2. Include `bitmovinplayer-analytics-conviva.js` **after** `conviva-core-sdk.min.js` in your HTML document

3. Create an instance of `ConvivaAnalytics` **before** calling `player.load(...)` and pass in your Conviva `CUSTOMER_KEY` and optional configuration properties:
    ```js
    var playerConfig = {
      key: 'YOUR-PLAYER-KEY',
      // ...
    };

    var container = document.getElementById('player');
    var player = new bitmovin.player.Player(container, playerConfig);
    
    // A ConvivaAnalytics instance is always tied to one player instance
    var conviva = new bitmovin.player.analytics.ConvivaAnalytics(player, 'CUSTOMER_KEY', {
      debugLoggingEnabled: true, // optional
      gatewayUrl: 'https://youraccount-test.testonly.conviva.com', // optional, TOUCHSTONE_SERVICE_URL for testing
    });
    
    var sourceConfig = {
      // ...
    };
    
    player.load(sourceConfig).then(function() {
      console.log('player loaded');
    }, function(reason) {
      console.error('player setup failed', reason);
    });
    ```

4. Release the instance by calling `conviva.release()` before destroying the player by calling `player.destroy()`
 
### Advanced Usage

#### VPF tracking

If you would like to track custom VPF (Video Playback Failures) events when no actual player error happens (e.g. 
the server closes the connection and return `net::ERR_EMPTY_RESPONSE` or after a certain time of stalling)
you can use following API to track those deficiencies.

```js
conviva.reportPlaybackDeficiency('Some Error Message', Conviva.Client.ErrorSeverity.FATAL);
```
_See [ConvivaAnalytics.ts](./src/ts/ConvivaAnalytics.ts) for parameter details._

Conviva suggests an timeout of about ~10 seconds and before reporting an error to conviva and providing feedback the user.

#### Content Metadata handling

If you want to override some content metadata attributes you can do so by adding the following:

```js
let metadataOverrides = {
  applicationName: 'App Name',
  viewerId: 'uniqueViewerId',
  custom: {
    customTag: 'customValue',
  },
  encodedFrameRate: 24,
  // … 
};

// …
// Initialize ConvivaAnalytics
// …

conviva.updateContentMetadata(metadataOverrides);
```

Those values will be cleaned up after the session is closed.

_See [ConvivaAnalytics.ts](./src/ts/ConvivaAnalytics.ts) for details about more attributes._

#### Consecutive playback

If you want to use the same player instance for multiple playback, just load a new source with `player.load(…)`.
The integration will close the active session.
 
```js
player.load({…});
```
