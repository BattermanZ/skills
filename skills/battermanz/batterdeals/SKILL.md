---
name: batterdeals
description: Find and live-test discount codes for an online shop, and report the shop's own savings (sale price, price policies, newsletter discount).
disable-model-invocation: true
---

# batterdeals

Hunt working discount codes for an online purchase, test them in a real cart, and report the full savings picture. Most shops have no working public code, so a run that ends "no code works, but the sale or a price policy saves you X" is a successful run, not a failed one.

The input decides the branch:

- **Product URL**: full run, steps 1 through 4.
- **Bare shop domain**: recon only, steps 1, 2 and 4. The report notes that cart-testing needs a product URL.

## Hard rules

These bind every step.

- Guest cart only. If checkout demands an account, stop and ask the user.
- The run ends with the best code applied and the cart left open for the user. Entering payment details or placing an order is never part of it.
- Test published codes only, meaning codes a source actually lists. Generating or pattern-guessing codes (SAVE10, SAVE15, ...) is forbidden: that is abusing the shop's promo system, not bargain hunting.
- Pace the cart: a few seconds between code attempts, at most 15 attempts per run.

## Step 1: orient

Extract the shop domain. On the product-URL branch, also fetch the product page and record product name, current listed price, and currency. That price is the baseline every measured discount is compared against.

Done when the domain is known and, on the product-URL branch, the baseline price is recorded.

## Step 2: search leg

Dispatch ONE background subagent to do all finding and vetting, so the aggregator noise stays out of this session and only a vetted candidate list comes back. Give it this task, placeholders filled in:

> Research discount codes and savings for SHOP_DOMAIN (product: PRODUCT_NAME at PRICE, if known). Work alone with your own web search and fetch tools; spawning agents or subagents is forbidden. Read pages only: no cart interaction, no form submission, no account creation.
>
> Cover all of these sources:
>
> 1. Caramel first pass: GET `https://grabcaramel.com/api/coupons?site=SHOP_DOMAIN&limit=50`. Unauthenticated JSON with server-side verification status and a lastWorkedAt timestamp, the highest-quality evidence available when it hits. Its catalog is US/UK-weighted, so an empty result or any failure is normal for EU shops: note it and move on.
> 2. Coupon aggregator sites via web search: "SHOP discount code" plus the shop's local-language equivalent (kortingscode, code promo, Gutscheincode, ...).
> 3. Deal communities: Reddit, the Pepper network (pepper.com, dealabs, ...), Slickdeals, and local equivalents.
> 4. The shop's own pages: does the product or its category appear in a sale or outlet section, and at what price? Do the terms or FAQ state code rules (stacking, exclusions)? Is there a price-match or lowest-price guarantee? Does newsletter signup advertise a discount? For all four, report only what the shop's own pages say, with the page you read it on.
>
> Vet every candidate; aggregators mostly publish junk. Evidence-rank the pool:
>
> - Per-deal timestamps, use counts, and server-side verification beat undated lists.
> - The same code set on several sites is ONE source, not several: aggregators plagiarize each other. Collapse them.
> - Discard codes a site itself admits are guesses ("codes from similar shops") and codes whose expiry lands within ~48h of today, a rolling-placeholder tell.
> - Click-to-reveal gated codes rank low unless a second independent source confirms them.
>
> Return three things: (1) up to 15 candidate codes, best evidence first, each with code, claimed discount, source, and the evidence for and against it; (2) the four shop-side answers, each with its source page; (3) a one-line verdict on how trustworthy the whole pool looks. Label aggregator claims as claims; call something verified only when the shop's own pages or server-side data back it.

Done when the subagent's report is back with every candidate carrying its evidence and all four shop-side answers present.

## Step 3: cart test (product-URL branch only)

Drive the shop with the Playwright browser tools, in front of the user.

1. Add the product to a guest cart and proceed until the promo field is found. It is often on the cart page rather than at checkout, collapsed behind a link ("kortingscode gebruiken", "add promo code").
2. Test candidates best-ranked first. Per attempt: enter the code, submit, read the outcome. Accepted: record the new cart total and the measured delta against the baseline. Rejected: record the shop's error message. Then clear the field or remove the applied code before the next attempt, clicking only the remove control inside the promo widget itself. A loosely matched "remove" selector hits the cart line items and silently empties the cart, a bug documented in prior art (Caramel's apply loop).
3. Stop early once a code delivers the largest discount any candidate claims; the rest cannot beat it.
4. Finish by applying the best measured code, or leaving the cart clean if none worked.

Done when every candidate has a verdict (accepted with measured delta, or rejected with the shop's error), or the 15-attempt cap is hit.

## Step 4: report

One chat message, nothing saved to disk or vault:

- Ranked table of candidates: code, claimed discount, verdict, measured delta, evidence. On the recon branch, claims and evidence only, plus the note that a product URL unlocks testing.
- The recommendation: best code and resulting price. When no code works, say so plainly and lead with the best shop-side alternative instead.
- Shop-side savings: sale price versus listed price, any price-match or lowest-price policy, and the newsletter flag ("shop advertises X% for signup; subscribing and fetching the code is on you").
