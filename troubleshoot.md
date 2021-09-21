# A place to list common troubleshooting scenarios

### Error adding key 

``` bash
$ docker trust signer add --key name.pub name localhost:5000
Adding signer "name" to localhost:5000...
Error: error contacting notary server: x509: certificate signed by unknown authority

Failed to add signer to: localhost:5000

```

Issue arises with docker not trusting the self signed certificate that comes imbedded into the notary repository. 