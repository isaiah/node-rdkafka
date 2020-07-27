```js
/*
 * node-rdkafka - Node.js wrapper for RdKafka C/C++ library
 *
 * Copyright (c) 2016 Blizzard Entertainment
 *
 * This software may be modified and distributed under the terms
 * of the MIT license.  See the LICENSE.txt file for details.
 */

var Kafka = require('../');


var oauthConf = 'principalClaimName=test principalName=test'
var producer = new Kafka.Producer({
  'metadata.broker.list': 'localhost:9092',
  'client.id': 'hey',
  'compression.codec': 'snappy',
  'security.protocol': 'SASL_PLAINTEXT',
  'sasl.mechanisms': 'OAUTHBEARER',
  'sasl.oauthbearer.config': oauthConf
});

producer.connect()
  .on('ready', function(i, metadata) {
    console.log(i);
    console.log(metadata);
  })
  .on('event.oauthbearer_token_refresh', function(e) {
      var token = {}; // TODO generate token
      var err = this.setOauthBearerToken();
      if (err) {
        this.setOauthBearerTokenFailure(err);
      }
  })
  .on('event.error', function(err) {
    console.log(err);
  });
```
