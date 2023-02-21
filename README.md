# ja3-spoofing-poc
An easy proof of concept of JA3 fingerprint spoofing

```js
const tls = require('tls');
const crypto = require('crypto');
const https = require('https');

// Define the JA3 fingerprint you want to spoof
const ja3 = "771,4865-4866-4867-49195-49199-49200-49201-49202-49205-49207-49300-49301-49302-49304-49305-49306-49491-49492-49493-49496-49497-49498-49499-57443-57603-57604-65281,0-23-24-25";

// Create a custom SSL/TLS context with the spoofed JA3 fingerprint
const context = tls.createSecureContext({
    ciphers: "ECDHE-RSA-AES128-GCM-SHA256",
    minVersion: "TLSv1.2",
    maxVersion: "TLSv1.3",
    getSecureContext: () => {
        const hash = crypto.createHash('md5').update(ja3).digest('hex');
        return tls.createSecureContext({
            key: fs.readFileSync('path/to/private/key'),
            cert: fs.readFileSync('path/to/certificate'),
            ALPNProtocols: ['http/1.1'],
            SNICallback: (hostname, cb) => {
                cb(null, tls.createSecureContext({
                    key: fs.readFileSync('path/to/private/key'),
                    cert: fs.readFileSync('path/to/certificate'),
                    ALPNProtocols: ['http/1.1'],
                    sessionTimeout: 600,
                    session: Buffer.from(hash, 'hex')
                }));
            }
        });
    }
});

// Use the custom context to make an HTTPS request with the spoofed JA3 fingerprint
https.request({
    hostname: 'example.com',
    port: 443,
    method: 'GET',
    path: '/',
    agent: new https.Agent({
        keepAlive: true,
        maxSockets: 50,
        maxFreeSockets: 10,
        secureContext: context
    })
}, (res) => {
    console.log(`Response status code: ${res.statusCode}`);
    // handle the response
}).end();
```
