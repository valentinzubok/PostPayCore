# PostPay — Studio Dev (chain 61997) deploy record

| | |
|---|---|
| **Network** | GenLayer Studio Dev / Studio Next — chain `61997` |
| **Contract** | [`0x037A89580b51b058F1B9163f1a8818dDd1e01363`](https://explorer-studio-dev.genlayer.com/address/0x037A89580b51b058F1B9163f1a8818dDd1e01363) |
| **Deploy tx** | [`0x0f592f11…5f7a31`](https://explorer-studio-dev.genlayer.com/tx/0x0f592f11c0a8711aeaee3ee57c06d4bbbc7357d5c8e46d05744a36c95f5c7a31) from `0x8E74c2362E776121282a3d9bB604c71e09f21FfE`, which has no role in the contract |
| **Source** | [`contracts/PostPay.py`](contracts/PostPay.py) — runner `py-genlayer:kzybg95x1gn21dw72m7hyg0maaap04n4nx55j9yt6fkdqxtz6bz0` |
| **Source sha256** | `cdc875a74cca1f5222c5f737e09215331df0d1d3d3f58f7c237953a5c31a93f0` |

The same file, byte-identical, is in both repositories: [PostPayCore](https://github.com/valentinzubok/PostPayCore)
(contract only) and [PostPay](https://github.com/valentinzubok/PostPay) (contract + console).

## Verify the source yourself

```bash
curl -s -X POST https://studio-dev.genlayer.com/api -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"gen_getContractCode","params":["0x037A89580b51b058F1B9163f1a8818dDd1e01363"]}' \
  | python3 -c "import sys,json,base64,hashlib; print(hashlib.sha256(base64.b64decode(json.load(sys.stdin)['result'])).hexdigest())"
shasum -a 256 contracts/PostPay.py
# both print cdc875a74cca1f5222c5f737e09215331df0d1d3d3f58f7c237953a5c31a93f0
```

CI runs the same comparison on every push (`scripts/verify_deployment.py`).

## Runner

The runner listed as current for `py-genlayer` on the
[official runners page](https://sdk.genlayer.com/main/executors/v0.4/python-sdk/available-runners.html) on 2026-11-24:
`py-genlayer:kzybg95x1gn21dw72m7hyg0maaap04n4nx55j9yt6fkdqxtz6bz0` (GenVM v0.4; the `# v0.3.0` header line is accepted).

## Lint

`sh scripts/lint_current_sdk.sh` — `genvm-lint lint` on the file as deployed, then `check` and `typecheck` on a copy
whose header uses the runner bundled with the public linter (`1jb45aa8…`); the linter cannot yet download the v0.4
runner. The contract body is byte-identical. Result on 2026-11-24: `Lint passed (3 checks)`, `Validation passed`
(20 methods: 14 view, 6 write), `No type errors found`.

## The run

All of it ran on 2026-11-24 as one state-aware script (it reads the chain before every step and waits for chain
time, not for a guess). Accounts are throwaway test keys:

* sponsor `0x6289823c2731D3c7cd8f548FF0361754A9C89D76`
* alice `0x2E36306F0a53b773229c4E1fc217936Fff57c5Ba`
* bob `0x873639DADe12C23763e718166822552D1eAEee29`
* carol `0x428436E98188EBB5f5139f748D93D83bdb45e7Cb`
* dave `0x9FAB80E13f374b9146363156a7DDF5F168cefdf3`
* stranger `0x8550B088Ee34b00B50Ed76022cDacaCF086efe50`

Alice, Bob, Carol and Dave are creators; the stranger runs every ruling round (anyone may).

The campaign `postpay-launch`: payout 1 GEN, max 2 paid posts (2 GEN escrowed), disclosure `#ad`, posts on
`httpbin.org`, 3 rounds per post (majority) 180 s apart, submission window 1500 s, ruling window 900 s
(submission deadline 2026-11-24 13:15:33 UTC, `1795526133`, final deadline 2026-11-24 13:30:33 UTC, `1795527033`). The brief, as validators saw it (`get_brief`):

1. Shows the disclosure "#ad" plainly, as a statement that the post is paid or sponsored (not negated, not hidden
   in a word, not only quoted from someone else) — added by the contract, always first
2. Links to the PostPay repository: https://github.com/valentinzubok/PostPay
3. Explains in the creator's own words what PostPay does for sponsors or creators

**Demo post pages.** The repository was not public during the run, so the post texts were served by
`https://httpbin.org/base64/<text>`, a public echo service that returns exactly the text encoded in its URL. The
texts are in [`fixtures/posts/`](fixtures/posts/) with the exact URLs used ([`fixtures/posts/README.md`](fixtures/posts/README.md));
decoding a URL gives the file. The frozen text and its sha-256 are on chain (`get_post`).

Every call is below, including the refused ones. Numbers and messages are copied from the script's log and chain reads.

| # | Call | Sender | Result | What happened | Tx |
|---|------|--------|--------|---------------|----|
| 1 | `open_campaign(postpay-wrong-value, PostPay launch posts) value 1.5 GEN` | sponsor | SUCCESS | open with the wrong value (1.5 GEN for 1 x 2) | [`0x9333d29a…b610be`](https://explorer-studio-dev.genlayer.com/tx/0x9333d29a4ef087ceae57dd6b1baf62a0eee9c5066c3b22f97a7d5fbc20b610be) |
| 2 | `open_campaign(postpay-launch, PostPay launch posts) value 2 GEN` | sponsor | SUCCESS | open the campaign | [`0x1a358d3c…d148a9`](https://explorer-studio-dev.genlayer.com/tx/0x1a358d3c6c4acbe8c332d8e1e8d6541da7d1c7bd790abbf4200c36711bd148a9) |
| 3 | `submit_post(postpay-launch, https://httpbin.org/ba…)` | alice | SUCCESS | alice submits | [`0xdaf74122…c14dc5`](https://explorer-studio-dev.genlayer.com/tx/0xdaf741222f8d12d14e77ff4ab5bf31de01a09467c3eb1c41cb07515804c14dc5) |
| 4 | `submit_post(postpay-launch, https://httpbin.org/ba…)` | sponsor | **ERROR** | the sponsor submits: **UserError** `the sponsor cannot submit a post` | [`0x0bea7296…981086`](https://explorer-studio-dev.genlayer.com/tx/0x0bea729667631894a34e903c15b61b1799ebeff448f50ab235de9964fb981086) |
| 5 | `submit_post(postpay-launch, https://httpbin.org/ba…)` | bob | SUCCESS | bob submits (wrong link) | [`0x17920cd9…4f8815`](https://explorer-studio-dev.genlayer.com/tx/0x17920cd9f1edb29b31cc9c3c9f7d251e9256b1066ade2522d1154517874f8815) |
| 6 | `submit_post(postpay-launch, https://httpbin.org/ba…)` | carol | SUCCESS | carol submits (no #ad) | [`0x4107a973…a7ef5e`](https://explorer-studio-dev.genlayer.com/tx/0x4107a9739b1007bb0d8fd6884522e1cf3987b26d572d340fde2b06e48fa7ef5e) |
| 7 | `submit_post(postpay-launch, https://httpbin.org/ba…)` | dave | **ERROR** | dave submits alice's URL: **UserError** `this post URL was already submitted as post 0` | [`0xb67b7694…e0bbaf`](https://explorer-studio-dev.genlayer.com/tx/0xb67b769454702aa368996b33f82d399eab9172b973063d0d19f374dfdde0bbaf) |
| 8 | `submit_post(postpay-launch, https://httpbin.org/ba…)` | dave | **ERROR** | dave submits a page without his claim code: **UserError** `the post does not contain your claim code pp-2e4b44ea4bfc4673 (see get_claim_code)` | [`0x0f85ae7b…2a4a08`](https://explorer-studio-dev.genlayer.com/tx/0x0f85ae7bc58248f5fb1e00943b048a30d8c70b46904535cf6b2139b7c22a4a08) |
| 9 | `submit_post(postpay-launch, https://example.com/dave-post)` | dave | **ERROR** | dave submits a post off the campaign site: **UserError** `post_url must be on httpbin.org` | [`0x64f8fc20…320633`](https://explorer-studio-dev.genlayer.com/tx/0x64f8fc20db57fab08f667a8db39dac907aa4e463202abe18be9516a5c2320633) |
| 10 | `submit_post(postpay-launch, https://httpbin.org/ba…)` | alice | **ERROR** | alice submits a second post: **UserError** `one post per address` | [`0x06d9d195…e5d2af`](https://explorer-studio-dev.genlayer.com/tx/0x06d9d195df3c41e5bcc4b3fce3136446daf03843b3c525314993fcf79de5d2af) |
| 11 | `submit_post(postpay-launch, https://httpbin.org/ba…)` | dave | SUCCESS | dave submits (never ruled) | [`0xf81f2ce3…39d6c6`](https://explorer-studio-dev.genlayer.com/tx/0xf81f2ce3265a910542397354a9d99f077820616d75b936e6f5983110ef39d6c6) |
| 12 | `rule_post(postpay-launch, 0)` | stranger | SUCCESS | rule post 0, round 1 → finding `judged`, elements [present, present, present], outcome `vote_approve` | [`0xd87d56b3…ff84b0`](https://explorer-studio-dev.genlayer.com/tx/0xd87d56b31654f1c5a4090753266b9fb7ec79af1e2d18faf30d07970eadff84b0) |
| 13 | `rule_post(postpay-launch, 0)` | stranger | **ERROR** | rule post 0 again before the interval: **UserError** `the next round for this post opens at 1795524925` | [`0x9e7ebbe1…033d37`](https://explorer-studio-dev.genlayer.com/tx/0x9e7ebbe1f868136b8bd005b843320f705eea4ff53fb07e6422100d1965033d37) |
| 14 | `rule_post(postpay-launch, 1)` | stranger | SUCCESS | rule post 1, round 1 → finding `judged`, elements [present, MISSING, present], outcome `vote_reject` | [`0x0b724513…08a49f`](https://explorer-studio-dev.genlayer.com/tx/0x0b7245134d3212f5f3cc69ca9bebc89d5d7fb7faa30c97d3aa47fbb5f408a49f) |
| 15 | `rule_post(postpay-launch, 2)` | stranger | SUCCESS | rule post 2 (decided in code) → finding `code` (disclosure_missing), elements [MISSING, -, -], outcome `final_rejected` | [`0x7cc6e5b7…19c14a`](https://explorer-studio-dev.genlayer.com/tx/0x7cc6e5b74c3c9af5c499771919dc227e0793122adbe21270369a3bf22919c14a) |
| 16 | `claim(postpay-launch, 0)` | bob | **ERROR** | bob claims alice's post: **UserError** `only the creator of this post may claim it` | [`0xf46f28de…45c4bd`](https://explorer-studio-dev.genlayer.com/tx/0xf46f28de269fad735e948b5c248c7c133b2fbc79582567cd67716eeda645c4bd) |
| 17 | `claim(postpay-launch, 0)` | alice | **ERROR** | alice claims before her post is decided: **UserError** `this post is not decided yet` | [`0x11a75c97…adcbb8`](https://explorer-studio-dev.genlayer.com/tx/0x11a75c97af37224364d6e4715fc71a24336a66827aba193a9bcc53fa94adcbb8) |
| 18 | `rule_post(postpay-launch, 0)` | stranger | SUCCESS | rule post 0, round 2 → finding `judged`, elements [present, present, present], outcome `final_approved` | [`0x2f41081e…cb9376`](https://explorer-studio-dev.genlayer.com/tx/0x2f41081ee184fec85aafa1bf93ad07344a34c101baf72ae02800a40216cb9376) |
| 19 | `rule_post(postpay-launch, 1)` | stranger | SUCCESS | rule post 1, round 2 → finding `judged`, elements [present, MISSING, present], outcome `final_rejected` | [`0x50136f27…ac6991`](https://explorer-studio-dev.genlayer.com/tx/0x50136f2731157ced5721baf5bf7d900c0b61a9ba74a85bf373bb66cac5ac6991) |
| 20 | `claim(postpay-launch, 1)` | bob | **ERROR** | bob claims his rejected post: **UserError** `this post was not approved` | [`0xe51699b8…270fc5`](https://explorer-studio-dev.genlayer.com/tx/0xe51699b8b631bc74c3bf91e78d7df37204c4616962054be67522d96a93270fc5) |
| 21 | `claim(postpay-launch, 0)` | alice | SUCCESS | alice claims her approved post | [`0xc7a42bcf…17666b`](https://explorer-studio-dev.genlayer.com/tx/0xc7a42bcf2edc30363a34975a5be8ffa7c272be3765be49d92580bd296217666b) |
| 22 | `refund_unspent(postpay-launch)` | sponsor | **ERROR** | the sponsor asks for the refund before the final deadline: **UserError** `the final deadline has not passed; the refund opens at 1795527033` | [`0xf5dcab8c…e5b703`](https://explorer-studio-dev.genlayer.com/tx/0xf5dcab8c41d97de412e6fa2f3f90ccdfbb5a7569c056bc376d893bb9f8e5b703) |
| 23 | `submit_post(postpay-launch, https://httpbin.org/ba…)` | stranger | **ERROR** | a late submission: **UserError** `submissions closed at 1795526133` | [`0xd0d25c25…1457ef`](https://explorer-studio-dev.genlayer.com/tx/0xd0d25c25b12073721d023863b96eb64b6b202cc1b65cabdb195cc118db1457ef) |
| 24 | `rule_post(postpay-launch, 3)` | stranger | **ERROR** | rule dave's post after the final deadline: **UserError** `the final deadline has passed at 1795527033; an unruled post counts as not approved` | [`0xa69fb711…d2b1ba`](https://explorer-studio-dev.genlayer.com/tx/0xa69fb711ece4807a133f49d2ae39962c98c4eea5923287e1fb99e90a42d2b1ba) |
| 25 | `claim(postpay-launch, 3)` | dave | **ERROR** | dave claims his unruled post: **UserError** `this post is not decided yet` | [`0xdc6213f6…5dde1a`](https://explorer-studio-dev.genlayer.com/tx/0xdc6213f6385ffb2ff9d8790687164dd390ffe258b318756ecc256435035dde1a) |
| 26 | `refund_unspent(postpay-launch)` | sponsor | SUCCESS | the sponsor takes the unspent budget back | [`0x660d1607…a0a677`](https://explorer-studio-dev.genlayer.com/tx/0x660d160779ed9f6745a5ccf09b0811bd2b32092c2d153408b31186e09ea0a677) |
| 27 | `withdraw_credit()` | sponsor | SUCCESS | the sponsor withdraws the credit | [`0x2e598633…da3233`](https://explorer-studio-dev.genlayer.com/tx/0x2e598633cbaba0feef80e6755364fca41c174898a16eaa3f3a04c1c45ada3233) |

### What the run shows

* **Approved and paid.** Post 0 (Alice): two agreed rounds 180 s apart, every element present, majority of 3
  reached → `final_approved`; Alice claimed 1 GEN (tx 21).
* **Rejected by validators.** Post 1 (Bob) links to another repository: both rounds found element 2 MISSING →
  `final_rejected`; Bob's claim was refused (tx 20).
* **Rejected in code.** Post 2 (Carol) has no `#ad`: the round was decided in code with no model call
  (`finding: code`, `disclosure_missing`), final in one round.
* **Unruled counts as not approved.** Post 3 (Dave) was valid but nobody ruled it; after the final deadline
  `rule_post` is refused (tx 24), the post shows `not_approved`, Dave cannot claim (tx 25), and its slot went back to
  the sponsor in the refund.
* **Whose post it is.** The sponsor could not submit (tx 4); Alice's URL could not be resubmitted by Dave (tx 7); a
  page without the submitter's claim code was refused (tx 8); a page off the campaign site was refused (tx 9); a
  second post by Alice was refused (tx 10).
* **Deadlines split time.** A ruling round before the 180 s interval was refused (tx 13); a submission after the
  submission deadline (tx 23), a round after the final deadline (tx 24) and the refund before it (tx 22) were each
  refused with their own message; the refund then worked (tx 26).
* **Money.** In: 2 GEN budget + 1.5 GEN of refused value (tx 1, credited). Out: 1 GEN to Alice (tx 21), 1 GEN unspent
  to the sponsor = payout x 1 unused slot (tx 26), 1.5 GEN credit back to the sponsor (tx 27). The contract holds 0.

### End state (read from chain after the last transaction)

`get_balance` → `{"balance":"0","credits":"0","held":"0"}`

`get_campaign("postpay-launch")`: `post_count 4`, `decided_count 3`, `approved_count 1`, `paid_count 1`,
`paid_out 1000000000000000000`, `refund_done true`, `refunded 1000000000000000000`.

`list_posts("postpay-launch", 0, 10)` payout states: post 0 `approved` / `paid` (2 rounds), post 1 `rejected` /
`rejected` (2 rounds), post 2 `rejected` / `rejected` (1 round, code), post 3 undecided / `not_approved` (0 rounds).

Every round is readable with `get_round("postpay-launch", i, k)`, with the post hash and per-element answers.

### What this run did not show

* **A round on which validators disagree.** Every ruling round in this run was recorded on the first try, so the
  "disagreement reverts, nothing is recorded, the round is sent again" path was not exercised on chain (the script
  checks the post's round count after every round and would have resent it). It is covered by unit tests
  (`test_a_consensus_failure_reverts_and_records_nothing`, `test_a_forged_agreed_answer_with_approved_not_matching_the_elements_reverts`).
* **An `unclear` round** (malformed or "unjudgeable" answer): a model cannot be made to fail on demand; covered by
  `test_a_malformed_or_unjudgeable_answer_counts_for_nothing` (11 cases).
* **The budget queue with two competing approved posts** (an approved post that is over budget, a claim waiting for
  an earlier post): covered by `test_the_budget_goes_in_submission_order_not_ruling_order`,
  `test_a_claim_waits_for_an_earlier_undecided_post`, `test_an_approved_post_after_the_budget_is_full_is_over_budget`.
* The refusal reason of the wrong-value campaign is in the call's return value, which the script did not log; the
  credit (1.5 GEN) is read from chain and withdrawn in the last transaction. The reason is covered by
  `test_the_value_must_equal_payout_times_max_posts`.
* Registry size and paging: unit tests (130 campaigns, 70 posts in one campaign), not hundreds of transactions.
