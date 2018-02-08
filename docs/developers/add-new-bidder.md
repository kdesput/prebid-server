# Adding a New Bidder

This document describes how to add a new Bidder to Prebid Server.

## Choose a Bidder Name

This name must be unique. Existing BidderNames can be found [here](../../openrtb_ext/bidders.go).

Throughout the rest of this document, substitute `{bidder}` with the name you've chosen.

## Define your Bidder Params

Bidders may define their own APIs for Publishers pass custom values. It is _strongly encouraged_ that these not
duplicate values already present in the [OpenRTB 2.5 spec](https://www.iab.com/wp-content/uploads/2016/03/OpenRTB-API-Specification-Version-2-5-FINAL.pdf).

Publishers will send values for these parameters in `request.imp[i].ext.{bidder}` of
[the Auction endpoint](../endpoints/openrtb2/auction.md). Prebid Server will preprocess these so that
your bidder will access them at `request.imp[i].ext.bidder`--regardless of what your `{bidder}` name is.

## Implement your Bidder

Bidder implementations are scattered throughout several files.

- `adapters/{bidder}/{bidder}.go`: contains an implementation of [the Bidder interface](../../adapters/bidder.go).
- `adapters/{bidder}/info.yaml`: contains contact info for the adapter's maintainer.
- `openrtb_ext/imp_{bidder}.go`: contract classes for your Bidder's params.
- `usersync/{bidder}.go`: A [Usersyncer](../../usersync/usersync.go) which returns cookie sync info for your bidder.
- `usersync/{bidder}_test.go`: Unit tests for your Usersyncer
- `static/bidder-params/{bidder}.json`: A [draft-4 json-schema](https://spacetelescope.github.io/understanding-json-schema/) which [validates your Bidder's params](https://www.jsonschemavalidator.net/).

Bidder implementations may assume that any params have already been validated against the defined json-schema.

## Test Your Bidder

### Automated Tests

Bidder tests live in two files:

- `adapters/{bidder}/{bidder}_test.go`: contains unit tests for your Bidder implementation.
- `adapters/{bidder}/params_test.go`: contains unit tests for your Bidder's JSON Schema params.

Since most Bidders communicate through HTTP using JSON bodies, you should
use the [JSON-test utilities](../../adapters/adapterstest/test_json.go).
This comes with several benefits, which are described in the source code docs.

If your HTTP requests don't use JSON, you'll need to write your tests in the code.
We expect to see at least 90% code coverage on each Bidder.

### Manual Tests

Build and start your server:

```bash
go build .
./prebid-server
```

Then `POST` an OpenRTB Request to `http://localhost:8000/openrtb2/auction`.

If at least one `request.imp[i].ext.{bidder}` is defined in your Request,
then your bidder should be called.

To test user syncs, [save a UID](../endpoints/setuid.md) using the FamilyName of your Usersyncer.
The next time you use `/openrtb2/auction`, the OpenRTB request sent to your Bidder should have
`BidRequest.User.BuyerUID` with the value you saved.

## Add your Bidder to the Exchange

Add a new [BidderName constant](../../openrtb_ext/bidders.go) for your {bidder}.
Update the [newAdapterMap function](../../exchange/adapter_map.go) to make your Bidder available in [auctions](../endpoints/openrtb2/auction).
Update the [NewSyncerMap function](../../usersync/usersync.go) to make your Bidder available for [usersyncs](../endpoints/setuid.md).

## Contribute

Finally, [Contribute](contributing.md) your Bidder to the project.