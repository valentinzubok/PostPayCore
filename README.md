# PostPay (contract)

<p align="center">
  <strong>Pay-per-post sponsorship: the budget is escrowed up front, validators check each post against a brief fixed at creation, and the first approved posts in submission order are paid.</strong>
</p>

<p align="center">
  <a href="https://github.com/valentinzubok/PostPayCore/actions/workflows/ci.yml"><img src="https://github.com/valentinzubok/PostPayCore/actions/workflows/ci.yml/badge.svg" alt="CI" /></a>
  <img src="https://img.shields.io/badge/GenLayer-Studio%20Dev%2061997-f472b6?style=flat-square" alt="Studio Dev" />
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue?style=flat-square" alt="MIT" /></a>
</p>

---

## The trust problem

A sponsor pays creators per post. A creator says "I posted what the brief asked"; the sponsor says
"the disclosure is missing", "wrong link", and does not pay. Today a middleman decides who gets paid,
or a single model reads the posts, and both are a party you have to trust: the middleman can side with
the sponsor who pays them, the model can be talked into "approved" by the post it is reading, and the
sponsor can change the brief after seeing the posts.

**PostPay escrows the whole budget before any post exists, fixes the brief, freezes each post whole on
chain, lets validators answer per element whether it is present, binds their consensus to the one bit
that moves money (approved or not), and pays in the order posts were submitted.** This repository is the
contract only; the console lives in [PostPay](https://github.com/valentinzubok/PostPay).

```
open_campaign(campaign_id, title, elements, disclosure, post_host, payout, max_posts,
              submission_window, ruling_window, rounds_required, round_interval)
                         PAYABLE, value = payout x max_posts (>= 1 GEN). No owner, no setters.
submit_post(campaign_id, post_url)
                         anyone but the sponsor, one post per address, strictly BEFORE the submission
                         deadline. The URL must be on post_host and not submitted before (any spelling);
                         the page (at most 2000 characters) is frozen whole and must contain the
                         creator's claim code (get_claim_code), so nobody can submit another's post
rule_post(campaign_id, index)
                         anyone, one round of one post, a full interval after its previous round,
                         strictly BEFORE the final deadline. No disclosure token in the text ->
                         rejected in code. Otherwise validators answer per element present/missing
                         and MUST AGREE ON `approved`; 1 or 3 rounds decide by majority, in code
claim(campaign_id, index)
                         the creator of an approved post. The first max_posts approved posts IN
                         SUBMISSION ORDER are paid; before the final deadline a claim waits until
                         every earlier post is decided, after it an unruled post counts as not approved
refund_unspent(campaign_id)
                         the sponsor, AT or after the final deadline: payout x the unused slots
withdraw_credit()        take back what open_campaign refused to escrow
get_schedule / list_posts(id, offset, limit) / get_round(id, i, k) / get_brief / get_claim_code
list_campaigns(offset, limit) / get_events(offset, limit)          paged, no global cap
```

## Risk table: what a sponsor or a creator can check before relying on it

| Risk | What stops it | Named tests |
|---|---|---|
| The sponsor does not pay | `payout x max_posts` is escrowed at `open_campaign`; the value must equal it exactly | `test_the_budget_is_escrowed_at_open`, `test_the_value_must_equal_payout_times_max_posts` |
| The sponsor changes the brief or the payout after seeing the posts | Every rule is fixed at creation; no owner, no setters; `rule_post` and `claim` take an id and an index only | `test_no_owner_and_no_setters`, `test_the_brief_is_stored_as_given_and_the_disclosure_comes_first`, `test_rule_takes_an_id_and_an_index_only` |
| Someone submits another creator's post as their own | The frozen text must contain the submitter's claim code, `sha256(campaign:address)`; another address has another code | `test_someone_elses_post_cannot_be_submitted_as_your_own`, `test_a_post_without_your_claim_code_is_refused`, `test_the_claim_code_differs_per_creator_and_per_campaign` |
| The same post is paid twice | A URL (in any spelling: case, trailing slash, fragment, `:443`) and an identical text are accepted once; one post per address | `test_a_duplicate_url_is_refused`, `test_a_duplicate_url_in_another_spelling_is_refused`, `test_the_same_text_under_a_new_url_is_refused`, `test_one_post_per_address` |
| A post hosted somewhere the sponsor did not ask for | The URL host must be `post_host` or a subdomain of it; `user@host` tricks and `..` segments are refused before any fetch | `test_a_post_off_the_campaign_site_is_refused`, `test_a_subdomain_of_the_campaign_site_is_accepted` |
| The sponsor pays themselves back through their own post | The sponsor's address cannot submit (a second wallet can: see "what you still trust") | `test_the_sponsor_cannot_submit` |
| The post is edited after it is judged | The page is frozen whole at submission (text + sha-256); rounds read only the frozen text | `test_the_post_is_frozen_whole_at_submission` |
| A verdict from part of a post | A post over 2000 characters, empty or unreadable is refused at submission; every accepted post is read whole | `test_a_post_longer_than_the_cap_is_refused`, `test_a_post_at_the_cap_is_accepted`, `test_an_unreadable_post_is_refused`, `test_an_empty_post_is_refused` |
| The post talks the model into "approved" | The post and the brief are fenced as untrusted data, fence markers scrubbed; instruction-like phrasing is flagged to the model and stored with the round | `test_the_post_is_fenced_as_untrusted_data`, `test_fence_markers_inside_a_post_are_scrubbed`, `test_injection_phrasing_is_flagged_and_stored_with_the_round` |
| Validators agree loosely and still disagree on whether the creator is paid | The consensus principle binds `approved`: one validator approving and another rejecting is a disagreement, the round reverts and records nothing. The contract re-checks `approved == all(elements)` | `test_the_principle_binds_approved`, `test_a_consensus_failure_reverts_and_records_nothing`, `test_a_forged_agreed_answer_with_approved_not_matching_the_elements_reverts`, `test_a_malformed_agreed_answer_reverts` |
| A malformed or evasive model answer decides | Only a JSON array of exactly one real boolean per element counts; `"true"`, `1`, a wrong length, `unjudgeable` or a model error is an `unclear` round that counts for nothing (kept as evidence) | `test_a_malformed_or_unjudgeable_answer_counts_for_nothing`, `test_approved_is_computed_in_code_from_every_element` |
| A model "finds" a disclosure that is not there | The disclosure token is checked in code over the whole frozen text; without it the post is rejected in code, no model call | `test_a_missing_disclosure_is_rejected_in_code_without_the_model`, `test_a_missing_disclosure_is_final_in_one_round_even_with_three_rounds`, `test_a_present_disclosure_still_goes_to_validators` |
| One odd reading decides a post | `rounds_required` = 3 decides by majority of agreed rounds a full interval apart, counted in code; an even number is refused | `test_three_rounds_decide_by_majority`, `test_two_agreeing_rounds_of_three_are_final`, `test_a_round_before_the_interval_is_refused`, `test_an_unclear_round_still_waits_a_full_interval` |
| Whoever is ruled first takes the budget | The queue is submission order: the first `max_posts` approved posts by index are paid. A claim before the final deadline waits until every earlier post is decided; the answer never changes once given | `test_the_budget_goes_in_submission_order_not_ruling_order`, `test_a_claim_waits_for_an_earlier_undecided_post`, `test_the_queue_answer_never_changes_once_given`, `test_a_rejected_earlier_post_frees_the_slot`, `test_an_approved_post_after_the_budget_is_full_is_over_budget` |
| An undecidable earlier post blocks payment forever | At the final deadline ruling stops and an unruled post counts as not approved, so every later claim resolves | `test_an_earlier_unruled_post_counts_as_not_approved_after_the_final_deadline`, `test_a_half_finished_three_round_post_is_not_approved_at_the_deadline`, `test_claims_after_the_final_deadline_still_pay` |
| Transaction order at the deadline chooses who is paid | Submit only `now < submission_deadline`; rule only `now < final_deadline`; refund only `now >= final_deadline`; deadline checked first, before any fetch. `get_schedule` derives from the same comparisons | `test_submission_at_or_after_the_deadline_is_refused`, `test_submission_one_second_before_the_deadline_is_accepted`, `test_ruling_at_or_after_the_final_deadline_is_refused`, `test_ruling_one_second_before_the_final_deadline_is_accepted`, `test_refund_one_second_before_the_final_deadline_is_refused`, `test_refund_at_the_final_deadline_is_accepted`, `test_the_schedule_never_offers_rule_and_refund_at_the_same_second`, `test_the_ruling_window_fits_every_round_of_a_late_post` |
| Money goes to someone else | Only the creator of an approved, in-budget post can claim, once; only the sponsor can take the unspent part, once, `payout x unused slots`. Books are updated before an external transfer | `test_only_the_creator_may_claim`, `test_a_post_is_paid_once`, `test_an_undecided_post_cannot_claim`, `test_only_the_sponsor_may_take_the_unspent_budget`, `test_the_unspent_budget_is_returned_once`, `test_the_refund_is_payout_times_the_unused_slots`, `test_money_leaves_only_to_the_sponsor_or_approved_creators` |
| A refused payable call swallows the money | `open_campaign` never raises once value is attached: the whole amount becomes a credit; `withdraw_credit` pays it back | `test_a_refused_campaign_never_raises_and_credits_everything` (18 cases), `test_a_malformed_elements_argument_is_refused_with_credit`, `test_a_taken_campaign_id_is_refused_with_credit`, `test_withdraw_credit_pays_back_once` |
| The evidence disappears | Every round is stored for good: post hash, per-element answers, `approved`, reason, flags, who and when (`get_round`); the frozen post stays on chain | `test_every_round_is_kept_with_its_evidence`, `test_events_record_each_step` |
| The registry fills up | No cap: `TreeMap`s with a `DynArray` index, at least 1 GEN per campaign, paged views | `test_many_campaigns_have_no_cap_and_are_paged`, `test_many_posts_in_one_campaign_are_paged`, `test_a_new_campaign_works_after_heavy_history`, `test_events_are_paged` |
| The books drift | balance == held + credits after every step of a 260-step random run, and 0 at the end | `test_the_books_balance_through_a_random_run` |

### What you still trust

* **The brief.** The sponsor writes it; validators judge exactly what it says. Read it (and `get_brief`) before you post.
* **Validators read honestly.** They must agree on `approved`; the per-element list stored with a round is the
  leading validator's explanation and may differ between validators in which element is missing.
* **Second wallets.** The sponsor's own address cannot submit, but a sponsor can post from another wallet. It still
  has to publish a post that meets the brief, on the declared site, before honest creators in the queue.
* **The post staying up.** The contract judges the post as it was at submission and does not check later that it
  was not deleted.
* **Queue speed.** A claim before the final deadline waits for every earlier post to be decided. A post nobody can
  judge delays later claims until the final deadline, never longer.

## Live

| | |
|---|---|
| Network | GenLayer Studio Dev / Studio Next — chain `61997` |
| Contract | [`0x037A89580b51b058F1B9163f1a8818dDd1e01363`](https://explorer-studio-dev.genlayer.com/address/0x037A89580b51b058F1B9163f1a8818dDd1e01363) — CI checks the deployed code equals [`contracts/PostPay.py`](contracts/PostPay.py) (sha-256 `cdc875a74cca1f5222c5f737e09215331df0d1d3d3f58f7c237953a5c31a93f0`) |
| Runner | `py-genlayer:kzybg95x1gn21dw72m7hyg0maaap04n4nx55j9yt6fkdqxtz6bz0` (GenVM v0.4, current on the official runners page) |
| Deploy record | [`STUDIO_DEV_DEPLOY.md`](STUDIO_DEV_DEPLOY.md) — the demo campaign `postpay-launch`: 27 transactions, 13 of them refused calls (sponsor submitting, duplicate URL, missing claim code, off-site post, second post, round before the interval, claim by a non-creator, claim of a rejected post, refund before and submission/ruling after the deadlines). One post approved and paid, one rejected by validators (wrong link), one rejected in code (no `#ad`), one never ruled; the unspent 1 GEN returned to the sponsor; contract balance 0. Raw log: [`docs/demo_run.log`](docs/demo_run.log) |

## Demo posts

The texts of the demo posts are in [`fixtures/posts/`](fixtures/posts/). During the on-chain run they were served by
`https://httpbin.org/base64/<text>`, a public echo service that returns exactly the text encoded in its URL (this
repository was not public yet, so `raw.githubusercontent.com` could not serve it). The campaign therefore declared
`httpbin.org` as its `post_host`. Each URL decodes to the fixture of the same name; the frozen text and its sha-256
are on chain (`get_post`). A real campaign declares the platform its creators post on (for example `x.com`).

## Lint

`sh scripts/lint_current_sdk.sh` runs the official GenVM linter (`lint`, `check`, `typecheck`); CI runs it on every
push. Every refusal raises `gl.vm.UserError`; every method takes `self`; helpers live at module level.

## Tests

```bash
pip install -r requirements-dev.txt
python3 -m pytest -q      # 131 tests (89 test functions, some parametrized)
```

[`tests/test_postpay.py`](tests/test_postpay.py) is grouped like the risk table above; every test it names exists
there.

## License

[MIT](LICENSE) © 2026 Valentyn Zubok
