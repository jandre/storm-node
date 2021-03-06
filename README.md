# storm-node

Simple implementation of [storm-multilang protocol](https://github.com/nathanmarz/storm/wiki/Multilang-protocol)
for nodeJS apps. Handles reading, emitting, acking, failing, and handshaking.

At this time, Bolts and Spouts listen to `process.stdin` and write to `process.stdout`; they are meant to be run
as standalone processes. To override this, change the `input` and `output` properties on your Bolt or Spout.

## Example

### Basic Bolt with automatic ack and anchoring

```javascript
'use strict';
var util = require('util');
var Storm = require('storm-node');

var TestBolt = function() {
  // Put any init code here.
};
// Inherit BasicBolt for automatic ack & anchoring.
util.inherits(TestBolt, Storm.BasicBolt);

TestBolt.prototype.process = function(tuple, done) {
  this.emit(["val1","val2"]);
  // `done` must be called to ack.
  done();
};

var bolt = new TestBolt();
bolt.run();
```

### Raw Bolt usage

```javascript
'use strict';
var util = require('util');
var Storm = require('storm-node');

var SplitSentenceBolt = function() {
  // Put any init code here.
};
util.inherits(SplitSentenceBolt, Storm.Bolt);

// Optional. If present will be called with storm configuration.
SplitSentenceBolt.prototype.onConfig = function(stormConfig) {

};

// Optional. If present, will be called with topology context.
SplitSentenceBolt.prototype.onContext = function(topologyContext) {

};

SplitSentenceBolt.prototype.process = function(tuple) {
  // Configuration is also available via `this.stormConfig`
  var words = tuple.tuple[0].split(" ");

  // Optionally, you can anchor this tuple. Emits sent after this line
  // will automatically have their `anchors` attribute set.
  this.anchoringTuple = tuple;

  for(var i = 0; i < words.length; i++)
  {
    this.emit([words[i]]);
  }
  // In a subclass of Storm.Bolt, `ack` must be called manually.
  this.ack(tuple);
  // Or fail.
  // this.fail(tuple);
};

var ssb = new SplitSentenceBolt();

// `run()` will start listening for messages via stdin.
ssb.run();
```

## Notes

`storm-node` exports four objects: 

```javascript
module.exports = {
  // Internal Communication library shared between Bolts and Spouts. 
  // You usually don't need to use this.
  Storm: Storm, 
    
  // A raw bolt. Similar to storm.Bolt in Java.
  // You need to manually ack when using this;
  // good for Bolts that emit more than once.
  Bolt: Bolt    

  // Similar to storm.BasicBolt. Automatically
  // acks on callback.
  BasicBolt: BasicBolt, 

  // WIP.
  Spout: Spout
}
```

## TODO

* Implement Spouts

## Author

Bryan Peterson - @lazyshot

## Contributors

@ssafejava
@jandre
