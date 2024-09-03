---
sidebar_position: 1
---

# Token payment process

This document describes the `/token/token_init` endpoint, which returns a response like the following example:

`https://api.mercurylayer.com/token/token_init`

```
{
  "btc_payment_address": "bc1qu2vs2z58m4dnyzuyk4240wdm2qd9jm827rnrjw",
  "fee": "50",
  "lightning_invoice": "lnbc939970n1pnvtm8ypp525ssrlm7senz7pf9z7h4sx7mz8nd2r7d6nxvldwjw7k2wm7hs8eqdp68psnxv3jxcekytf5x5cnwtf5xp3rwttzxpskxttzxumrgwpc8yer2cej8qcqzzsxqzjcsp5rd992e42ff4my6safcjce3gs0cz74rsuyffhqpragclzzkfw0lcq9qxpqysgq83hpdfxvsgh6ve8afvj38f2uuhg6nzm042sz98ge4qknfrcurlr8n3prw5qtmvjvcc7lpkhrfvm09ex5q3v6aq3tsprq958fakkrljgp2zfy80",
  "processor_id": "bb038002-b7cf-49e7-8425-da4d28e8242e",
  "token_id": "8a32263b-4517-40b7-b0ac-b76488925c28"
}
```

The fee is provided in euros. The invoice can be paid via Lightning Network (LN). To get additional details or to pay on-chain, make a call to:

`https://api.swiss-bitcoin-pay.ch/checkout/bb038002-b7cf-49e7-8425-da4d28e8242e`

with the `processor_id`. This will return the on-chain payment amount (in millisatoshis), as shown in the example below:

```
{
  "id": "bb038002-b7cf-49e7-8425-da4d28e8242e",
  "amount": 93997000,
  "wallet": "041feacad5e94ead9cf1e6b6b3acda14",
  "time": 1724247268,
  "expiry": 1724250868,
  "grossAmount": null,
  "title": "8a32263b-4517-40b7-b0ac-b76488925c28",
  "description": null,
  "status": "open",
  "input": {
    "unit": "EUR",
    "amount": 50
  },
  "paymentDetails": [
    {
      "network": "lightning",
      "paymentRequest": "lnbc939970n1pnvtm8ypp525ssrlm7senz7pf9z7h4sx7mz8nd2r7d6nxvldwjw7k2wm7hs8eqdp68psnxv3jxcekytf5x5cnwtf5xp3rwttzxpskxttzxumrgwpc8yer2cej8qcqzzsxqzjcsp5rd992e42ff4my6safcjce3gs0cz74rsuyffhqpragclzzkfw0lcq9qxpqysgq83hpdfxvsgh6ve8afvj38f2uuhg6nzm042sz98ge4qknfrcurlr8n3prw5qtmvjvcc7lpkhrfvm09ex5q3v6aq3tsprq958fakkrljgp2zfy80",
      "hash": "VSEB/36GZi8FJRevWBvbEebVD83UzM+10nesp2/XgfI="
    },
    {
      "network": "onchain",
      "address": "bc1qu2vs2z58m4dnyzuyk4240wdm2qd9jm827rnrjw"
    }
  ],
  "device": {},
  "extra": {
    "originalHash": "bb038002-b7cf-49e7-8425-da4d28e8242e",
    "tag": "invoice-web"
  },
  "createdAt": 1724247268,
  "delay": 3600,
  "pr": "lnbc939970n1pnvtm8ypp525ssrlm7senz7pf9z7h4sx7mz8nd2r7d6nxvldwjw7k2wm7hs8eqdp68psnxv3jxcekytf5x5cnwtf5xp3rwttzxpskxttzxumrgwpc8yer2cej8qcqzzsxqzjcsp5rd992e42ff4my6safcjce3gs0cz74rsuyffhqpragclzzkfw0lcq9qxpqysgq83hpdfxvsgh6ve8afvj38f2uuhg6nzm042sz98ge4qknfrcurlr8n3prw5qtmvjvcc7lpkhrfvm09ex5q3v6aq3tsprq958fakkrljgp2zfy80",
  "isPaid": false,
  "isExpired": false,
  "hash": "bb038002-b7cf-49e7-8425-da4d28e8242e",
  "fiatAmount": 50,
  "fiatUnit": "EUR",
  "onChainAddr": "bc1qu2vs2z58m4dnyzuyk4240wdm2qd9jm827rnrjw",
  "minConfirmations": 1,
  "isPending": false
}
```

Once the payment is made, calling the `/token/token_verify/<token_id>` endpoint will return `true` or `false`. If `true`, the `token_id` can then be used with the client deposit functions.