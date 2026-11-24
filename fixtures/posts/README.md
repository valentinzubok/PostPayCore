# Demo posts

The texts the on-chain demo (`STUDIO_DEV_DEPLOY.md`) read. Each was served at
`https://httpbin.org/base64/<the text, url-safe base64>`, a public echo service that returns exactly the encoded text,
because this repository was not public at the time of the run. Decode any URL below and it equals the file.

| File | Role in the demo | Chars | URL used on chain |
|---|---|---|---|
| [`alice.md`](alice.md) | post 0 - every element present: approved, paid | 315 | `https://httpbin.org/base64/I2FkIFBhaWQgcG9zdC4gSSB0cmllZCBQb3N0UGF5IHRoaXMgd2Vlay4gQSBzcG9uc29yIGxvY2tzIHRoZSB3aG9sZSBidWRnZXQgaW4gZXNjcm93IGJlZm9yZSBhbnlvbmUgcG9zdHMsIGFuZCBHZW5MYXllciB2YWxpZGF0b3JzIGNoZWNrIGV2ZXJ5IHBvc3QgYWdhaW5zdCB0aGUgYnJpZWYsIHNvIGEgY3JlYXRvciB3aG8gZGlkIHdoYXQgd2FzIGFza2VkIGdldHMgcGFpZCB3aXRob3V0IGFyZ3Vpbmcgd2l0aCBhbnlvbmUuIFJlcG86IGh0dHBzOi8vZ2l0aHViLmNvbS92YWxlbnRpbnp1Ym9rL1Bvc3RQYXkgLSBjbGFpbSBwcC1mOWQ3ZDM0YzJhM2U1NGIw` |
| [`bob.md`](bob.md) | post 1 - links to a different repository: rejected by validators (element 2 missing) | 190 | `https://httpbin.org/base64/I2FkIFNwb25zb3JlZC4gUG9zdFBheSBsZXRzIGEgc3BvbnNvciBlc2Nyb3cgYSBidWRnZXQgYW5kIHBheXMgZWFjaCBjcmVhdG9yIHdob3NlIHBvc3QgdGhlIHZhbGlkYXRvcnMgYXBwcm92ZS4gTW9yZSBoZXJlOiBodHRwczovL2dpdGh1Yi5jb20vZXhhbXBsZS9vdGhlci1wcm9qZWN0IC0gY2xhaW0gcHAtMjQ5ZTM4NzA4OTViMDA0OQ==` |
| [`carol.md`](carol.md) | post 2 - no "#ad": rejected in code | 198 | `https://httpbin.org/base64/SSB0cmllZCBQb3N0UGF5OiB0aGUgc3BvbnNvcidzIGJ1ZGdldCBzaXRzIGluIGVzY3JvdyBhbmQgdmFsaWRhdG9ycyByZWFkIGVhY2ggcG9zdCBhZ2FpbnN0IHRoZSBicmllZiBiZWZvcmUgYW55b25lIGlzIHBhaWQuIFJlcG86IGh0dHBzOi8vZ2l0aHViLmNvbS92YWxlbnRpbnp1Ym9rL1Bvc3RQYXkgLSBjbGFpbSBwcC00YmZhOTc3YmExNTg3Yzdj` |
| [`dave.md`](dave.md) | post 3 - valid, but nobody ruled it before the final deadline: counts as not approved | 213 | `https://httpbin.org/base64/I2FkIFBhaWQgcGFydG5lcnNoaXAuIFBvc3RQYXkgcGF5cyBjcmVhdG9ycyBwZXIgcG9zdDogdGhlIGJ1ZGdldCBpcyBlc2Nyb3dlZCB1cCBmcm9udCBhbmQgdmFsaWRhdG9ycyBkZWNpZGUgaWYgYSBwb3N0IG1hdGNoZXMgdGhlIGJyaWVmLiBTb3VyY2U6IGh0dHBzOi8vZ2l0aHViLmNvbS92YWxlbnRpbnp1Ym9rL1Bvc3RQYXkgLSBjbGFpbSBwcC0yZTRiNDRlYTRiZmM0Njcz` |
| [`sponsor.md`](sponsor.md) | refused: the sponsor cannot submit | 123 | `https://httpbin.org/base64/I2FkIE91ciBvd24gcG9zdCBhYm91dCBQb3N0UGF5LCBwYWlkIGZvciBieSBvdXJzZWx2ZXMuIGh0dHBzOi8vZ2l0aHViLmNvbS92YWxlbnRpbnp1Ym9rL1Bvc3RQYXkgLSBjbGFpbSBwcC02NDQ4NmRlZWEzZTE0ODRi` |
| [`nocode.md`](nocode.md) | refused: the page does not contain the submitter's claim code | 140 | `https://httpbin.org/base64/I2FkIERhdmUncyBzZWNvbmQgdGFrZSBvbiBQb3N0UGF5LCBhIHBheS1wZXItcG9zdCBlc2Nyb3cganVkZ2VkIGJ5IHZhbGlkYXRvcnMuIGh0dHBzOi8vZ2l0aHViLmNvbS92YWxlbnRpbnp1Ym9rL1Bvc3RQYXkgKG5vIGNsYWltIGNvZGUgaGVyZSk=` |
| [`alice2.md`](alice2.md) | refused: one post per address | 103 | `https://httpbin.org/base64/I2FkIEFub3RoZXIgUG9zdFBheSBwb3N0IGJ5IEFsaWNlLiBodHRwczovL2dpdGh1Yi5jb20vdmFsZW50aW56dWJvay9Qb3N0UGF5IC0gY2xhaW0gcHAtZjlkN2QzNGMyYTNlNTRiMA==` |
