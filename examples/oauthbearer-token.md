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

/*
now := time.Now()
nowSecondsSinceEpoch := now.Unix()

// The token lifetime needs to be long enough to allow connection and a broker metadata query.
// We then exit immediately after that, so no additional token refreshes will occur.
// Therefore set the lifetime to be an hour (though anything on the order of a minute or more
// would be fine).
expiration := now.Add(time.Second * time.Duration(3600))
expirationSecondsSinceEpoch := expiration.Unix()

oauthbearerMapForJSON := map[string]interface{}{
        principalClaimName: principal,
        "iat":              nowSecondsSinceEpoch,
        "exp":              expirationSecondsSinceEpoch,
}
claimsJSON, _ := json.Marshal(oauthbearerMapForJSON)
encodedClaims := base64.RawURLEncoding.EncodeToString(claimsJSON)
jwsCompactSerialization := joseHeaderEncoded + "." + encodedClaims + "."
extensions := map[string]string{}
oauthBearerToken := kafka.OAuthBearerToken{
        TokenValue: jwsCompactSerialization,
        Expiration: expiration,
        Principal:  principal,
        Extensions: extensions,
}
*/
var generateJWT = function() {
  var now = new Date();
  var nowSeconds = now.getTime();
  now.setHours(now.getHours() + 1);
  var expiration = now.getTime();
  var principalClaimName = 'sub';
  var principal = 'test';

  var oauthBearerMapForJSON = {
    [principalClaimName]: principal,
    'iat': nowSeconds,
    'exp': expiration
  };

  var claimsJSON = JSON.stringify(oauthbearerMapForJSON);
  var encodedClaims = Buffer.from(claimsJSON).toString('base64').replace('+', '-').replace('/', '_').replace(/=+$/, '');
  var token = secret + '.' + encodedClaims + '.'; // TODO replace with the specific algorithm used by your cluster

  var jwt = {
    'tokenValue': token,
    'expiration': expiration,
    'principal': principal,
    'extensions': {},
  };
  return jwt;
}

producer.connect()
  .on('ready', function(i, metadata) {
    console.log(i);
    console.log(metadata); })
  .on('event.oauthbearer_token_refresh', function(e) {
      var token = generateJWT();
      var err = this.setOauthBearerToken(token);
      if (err) {
        this.setOauthBearerTokenFailure(err);
      }
  })
  .on('event.error', function(err) {
    console.log(err);
  });
```
