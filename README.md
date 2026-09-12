## Auction Bot Plus

An auction house bot module for **AzerothCore, World of Warcraft: Wrath of the Lich King 3.3.5a**. This fork provides a customized default configuration, with the goal of giving administrators understandable control over auction listings and pricing through configuration files.

Feature highlights:

- Configuration-file control of bot behavior, without SQL-based tuning tables.
- Item inclusion/exclusion rules, category and quality listing proportions, and stack-size settings.
- Explicit item-price overrides or runtime formula pricing, with configurable random variations.
- Optional advanced pricing rules for supported item categories.
- Separate seller and buyer settings.
- Multiple bot characters for different seller names on auctions.
- GM and server-console commands for configuration reloads, update calls and removal of bot-owned auctions.

## Requirements

Use a compatible AzerothCore WotLK core and its required build dependencies.

Upstream documentation identifies [change set 3f46e05](https://github.com/azerothcore/azerothcore-wotlk/commit/3f46e05d3691895b6b8a5b3832d17ecb1e210791) as a historical minimum. This is not a guarantee that every later core revision is compatible with the current module; check the module's required hooks when choosing a core revision.

## Installation

1. Place the module under the `modules` directory of your AzerothCore source.
2. Re-run CMake, rebuild AzerothCore and install the resulting server/module files using your normal build procedure.
3. Create or update the active `mod_ahbot.conf` from the supplied `conf/mod_ahbot.conf.dist`.
4. Create at least one dedicated bot character and set `AuctionHouseBot.GUIDs` to its real character GUID. See **Usage** below.

**Existing installations:** back up your active configuration, compare it with the new `.conf.dist`, and merge the desired settings. Preserve your real bot GUIDs and local customizations. Do not assume a rebuild replaces your active config, and do not overwrite working GUIDs with the distributed value of `0`.

## About this fork

This fork ships a customized default config containing market-associated price inputs, stored formula estimates and manually adjusted overrides. Items without an explicit override use the module's runtime pricing formula.

The checked module source is unmodified relative to upstream revision [`f685832`](https://github.com/NathanHandley/mod-ah-bot-plus/commit/f685832994c825f90aa5a3dc0e1620aa568e875b). No optional vendor-guard source patch is assumed in the documentation's calculations.

### Upstream vs this fork

The comparison below uses that upstream revision. Database-dependent counts refer to the checked **46,096-template** AzerothCore world baseline, revision `16685343110115b12e76d517f7bac15c6a97fb2a`, not a live inventory on every server.

| Setting | Upstream comparison | This fork |
|---|---:|---:|
| Price override system | off | on |
| Explicit item-price overrides | 0 | **12,906** |
| Configured blacklist IDs, with ranges expanded | 2,419 | **33,053** |
| Blacklist IDs present in the checked world database | 2,402 | **33,050** |
| Non-blacklisted item templates, before other seller filters | 43,694 | **13,046** |
| Other config entries differing by text comparison | — | **84** |

The 84 additional differences exclude the price and blacklist lists. Some are formatting differences rather than behavioral changes. **52 `ListProportion` entries** differ in this comparison.

Non-blacklisted does not mean listable. In the checked 13,046-template set, another **1,026** fail tested seller filters and **162** have zero category-quality listing weight. Passing those stages still does not guarantee generation of a listing.

### What changed

**Pricing** — 12,906 explicit item-price overrides, classified in the accompanying price list as:

| Label | Count | Interpretation |
|---|---:|---|
| **S** | 5,299 | Inherited price inputs; prior statistical comparison suggests a Warmane/Lordaeron connection, not confirmed collection metadata. |
| **W** | 3,345 | Imported/calibrated inputs attributed to AuctionSim/Auctioneer data associated with Warmane/Lordaeron; the full collection chain has not been independently authenticated. |
| **P** | 4,255 | Formula-estimated amounts saved as explicit overrides, not recomputed through the formula on each evaluation. |
| **A** | 7 | Adjusted explicit overrides intended to reduce vendor-resale margins; not proof that all exploit routes are closed. |

S and W total **8,644**, approximately **67% of the overrides**. These are source classifications, not a claim that 67% of prices are authenticated current market observations. The exact original collection dates and complete provenance are unverified. Prices may not suit your server's economy.

An additional **140 non-blacklisted templates have no explicit override** and are documented in the runtime supplement to the Complete Item Price List. This brings that document's coverage to **13,046 templates**. It is not a catalogue of everything the buyer could purchase: seller-blacklisted items can still receive buyer valuations.

**Item pool** — the config explicitly blacklists 33,053 IDs. Of the 33,050 present in the checked database, **20,656 are bind-on-pickup** and **3,954 are quest-bound**. Those bindings normally prevent player auctions. Other exclusions include poor-quality items, internal/test-marked templates, pigments, containers, vendor-backed items, scrolls and additional catalogue choices. Item properties do not always establish the original reason for an exclusion or prove an exploit was closed. Three configured IDs are absent from this baseline.

**Auction house volume** — the config requests higher listing thresholds than the upstream comparison, while retaining a **smaller non-blacklisted template pool**:

| Auction house | MinItems | MaxItems |
|---|---:|---:|
| Alliance | 46,000 | 52,000 |
| Horde | 43,000 | 48,000 |
| Neutral | 28,000 | 32,000 |
| Total for separate houses | **117,000** | **132,000** |

`MinItems` is the threshold below which the seller attempts to add auctions. Once a house is at or above that minimum, the seller stops adding; it does not automatically fill to `MaxItems`. A batch may take the count above the minimum. Counts include player auctions, not only bot listings. With cross-faction auctions enabled, the module uses the neutral pass instead of separate Alliance and Horde passes.

**Buyer settings** — this fork uses `AcceptablePriceModifier = 1.0` and `AlwaysBidMaxCalculatedPrice = false`. The checked upstream config already uses the equivalent modifier **1** and the same **false** setting, so these are **not behavioral changes relative to that upstream revision**. One actual difference is `BidAgainstPlayers`: **false upstream, true in this fork**, allowing the buyer to consider auctions with non-bot player bids under its normal rules. A buyout pays its posted price, not automatically the budget ceiling. These settings do not eliminate every profitable resale route.

**Population time** — `ItemsPerCycle = 120` sets the maximum requested batch of new auctions per applicable seller-house invocation; actual successful generation can be lower. `MinutesBetweenSellCycle = 1:2` controls seller interval counters. Actual fill time depends on update timing, successful item generation, house counts and other rules; there is no guaranteed “few hours” fill time. `.ahbot update` runs one normal update call, not an unconditional refill.

**Exploit mitigation and remaining exposure** — seller exclusions and price adjustments should not be described as closing all vendor, milling or container resale loops. The seller blacklist prevents new bot-generated listings of excluded IDs; it is **not a buyer denylist** and does not automatically remove existing auctions.

Blacklisting selected containers reduces their bot-generated supply but does not establish that every alternative source or profitable conversion is gone. Blacklisting pigments does not prevent players milling herbs and selling the outputs to the buyer. The audit found a conditional **Silverleaf-to-Alabaster-Pigment** resale example under this config.

Using the checked unmodified module, undiscounted vendor costs and the audit's **5% AH-cut assumption**, the expanded vendor audit identifies **108 potential vendor-to-bot resale opportunities**: **102** have positive margins at prices below the lowest checked buying budget, and **6** require favorable buyer valuations. These include items that can also be crafted or farmed. Availability, acquisition conditions, auction eligibility and completed purchases still matter; these are not guaranteed earnings.

See the [Exploitable Vendor Items List](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Exploitable_Vendor_Items.pdf) for prices, vendor sources, stock, restock calculations and limitations.

### Documentation

The `Ah Bot Plus V.2 PDFs/` folder contains supporting documentation:

| File | Contents |
|---|---|
| [Complete Item Price List](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Plus_V2_Complete_Item_Price_List.pdf) | **255 pages.** The 13,046 checked non-blacklisted templates: 12,906 explicit overrides and 140 runtime entries. Includes calculated buyout/bid ranges, one-item gross allowances, source labels and seller/eligibility caveats. These are not guaranteed payouts or net profits. |
| [Do Not Sell List](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Do_Not_Sell_List.pdf) | **52 pages.** All 2,467 positive vendor-guard caps from the checked lookup, with whole-auction limits, bound/timed flags, stack examples and explanations of why addon suggestions do not guarantee bot purchases. |
| [Selling To The Bot](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Selling_To_The_Bot.pdf) | **10 pages.** Corrected player guide covering minimum-budget pricing, stacks, fees, bids, candidate selection and remaining checks. Includes the Iron Riveted War Helm example on page 1 and the remaining-checks checklist on page 9. No guaranteed-sale or fixed-wait-time claims. |
| [Config Revision Comparison](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Config_Revision_Comparison.pdf) | **11 pages.** Developer changelog comparing the previous and current config revisions. The “New” config corresponds to the current mod_ahbot.conf.dist; other filenames identify historical developer revisions. The original comparison contains inaccurate or outdated claims. Read the appended correction supplement alongside it for corrected payout calculations, vendor-opportunity scope, restock mechanics, mitigation advice and profession-related conclusions. |
| [Exploitable Vendor Items List](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Exploitable_Vendor_Items.pdf) | **106 pages.** Analysis of 108 potential vendor-to-bot resale opportunities: 102 floor-positive and 6 requiring favorable buyer valuations. Includes 3,688 vendor-source records, stock/restock details, costs and conditional net margins, without an optional guard patch. |
| [Complete Blacklist & 72 Added Items](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Complete_Blacklist_&_72_Added_Items.pdf) | **652 pages.** Full 33,053-ID blacklist and the 72 additions, with explanations of why those additions alone do not prove exploit closure. Config filenames inside the document identify the compared developer revisions. |
| [How the Bot Decides to Buy — Rakzur Club Example](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Rolled_Valuation_Rakzur_Club_Guide.pdf) | **3 pages.** A walkthrough of how the bot decides whether to buy an item a player has auctioned, using Rakzur Club as an example. Explains its random buying budget, how your asking price affects the decision, and why a purchase is not guaranteed. |
| [How the Bot Decides to Buy — Stacked Items](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Rolled_Valuation_Stacked_Items_Guide.pdf) | **9 pages.** A walkthrough of how the bot decides whether to buy stacks of items from player auctions. Covers all four pricing categories, with examples showing how stack size, asking price, random valuation and vendor caps affect the decision. |
| [Vendor-Guard Stack Quirk — Known Bug](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_VendorGuard_StackQuirk_KnownBug.pdf) | **3 pages.** The buyer compares a stack's total price against the per-unit vendor cap, so fairly priced stacks are refused whenever the total exceeds the single-unit cap. Covers the Runic Healing Potion example, practical consequences of the bug, and the staged (not installed) fix on the unpatched build. |
| [Excel workbooks](AH%20Bot%20Plus%20V.2%20Excel%20Files) | This folder will contains the Complete Item Price List and the Do Not Sell List as Excel Files. |


If you find discrepancies, open an issue with the item ID, relevant settings and document page so the result can be checked.

### Known open issues

- **Vendor-guard stack bug**. The AH BOT (buyer) compares a stack's **total** price against the **per-unit** vendor cap (`src/AuctionHouseBot.cpp`, lines 1764/1769), so a fairly priced stack is refused whenever its total exceeds the single-unit cap — e.g. Runic Healing Potion (33447, 60s cap) passes as a single at 60s, but a 20-stack at 60s each (12g total) is refused. Singles, seller pricing, and flip-blocking are unaffected: the guard only refuses and never overpays if the item is contain within the [Do Not Sell List](Ah%20Bot%20Plus%20V.2%20PDFs/AHBot_Do_Not_Sell_List.pdf). 
- **Vendor resale remains possible.** The 108-item audit is conditional, not a promise of sales or income. The existing guard excludes class-7 trade goods, and zero-SellPrice entries bypass its check. Seller blacklisting does not block the buyer from purchasing player auctions.
- **Conversion exploits are not certified closed.** Pigment and container exclusions alone do not establish that milling, opening containers or alternative acquisition routes are unprofitable.
- **Buyer/seller price overlap needs further review.** For override-priced items, seller starting bids can reach approximately **0.60×** the stored value, while a separately calculated buyer budget can reach approximately **1.15×**, before rounding and other restrictions. Overlapping buyout valuations also matter; removing bidding is not the only possible mitigation.
- **Crafting and disenchanting profitability have not been fully audited.** A complete assessment requires the relevant recipe/spell, item and loot data, core eligibility rules and server-specific changes. Neither system should be described as exploit-free without that analysis.
- **`MaxBuyoutPriceInCopper` is not a universal auction-total cap.** Explicit overrides return before the runtime clamp, and the seller multiplies unit values by stack count afterward. Do not assume every bot auction is limited to 100,000g. For example, the checked Wooly White Rhino override can produce a one-item valuation of **109,244g 82s 56c**.

Buyer-side guard extensions and conversion-aware pricing are separate mitigation options, each with trade-offs for legitimate player sales. No optional 15-item vendor-exclusive patch is assumed installed.

## Usage

Create dedicated, ordinary characters for the auction bot. Set `AuctionHouseBot.GUIDs` to their character GUIDs from the characters database. Their names will appear on auctions. If using playerbots or a similar module, do not use characters managed by that module as the auction bot characters.

The distributed config has:

```ini
AuctionHouseBot.GUIDs = 0
AuctionHouseBot.EnableSeller = true
AuctionHouseBot.Buyer.Enabled = true
AuctionHouseBot.ItemsPerCycle = 120
AuctionHouseBot.MinutesBetweenSellCycle = 1:2
AuctionHouseBot.MinutesBetweenBuyCycle = 5:20
AuctionHouseBot.Buyer.BuyCandidatesPerBuyCycle = 1:3
AuctionHouseBot.Buyer.AcceptablePriceModifier = 1.0
AuctionHouseBot.Buyer.AlwaysBidMaxCalculatedPrice = false
AuctionHouseBot.Buyer.PreventOverpayingForVendorItems = true
AuctionHouseBot.Buyer.BidAgainstPlayers = true
```

**Replace `GUIDs = 0` before expecting bot activity.** Seller and buyer are already enabled in this fork's config. The bot accounts do not require GM privileges. Keep the dedicated characters out of normal player use; logging them in to browse the auction house can cause conflicts.

### Pricing configuration

- **Explicit override path:** the 12,906 stored amounts bypass category, quality, category-quality, item-level and advanced formula multipliers. Enabled override buyout/bid variations still apply, as do buyer-side settings and guards.
- **Runtime formula path:** category, quality, category-quality and advanced multipliers contribute to the calculation. For example, multipliers of 1.5, 2 and 1.4 nominally combine to 4.2, although stepwise floating-point operations and integer conversion affect exact copper results. The item-level multiplier is used only when its source-code conditions hold, including a computed advanced multiplier of exactly `1.0`; an enable flag alone is not the full test.
- **Rounding and limits:** the code uses integer prices and floating-point operations. For exact thresholds, use the checked copper values rather than rounded percentages. The runtime clamp is not an override or whole-stack price ceiling.

### In-Game Commands

The following commands require GM access in game and are also registered for the server console:

| Command | Description |
|---|---|
| `.ahbot reload` | Reloads module configuration and rebuilds the relevant candidate lists/settings. It does not automatically reprice existing auctions. |
| `.ahbot empty` | Removes bot-owned auctions and attempts to refund affected bidders through the normal removal path. Player-owned auctions are not the target. **Use with caution.** |
| `.ahbot update` | Runs one normal update call and advances cycle counters. Buying or selling occurs only when its cycle is due and the other rules permit it. Repeated calls may be needed; this is not an unconditional refill. |
| `.ahbot help` | Shows command help. |

## Buying Bot Behavior

### 1. Activity and candidate selection

The buyer must be enabled and have valid configured bot characters. The current buy interval setting is **5:20**, and candidate attempts are **1:3 per applicable buyer-house invocation**. The source advances counters on update calls; manual `.ahbot update` calls also advance them. These settings do not guarantee a purchase every minute or an exact time-to-sale.

The candidate query excludes auctions owned by configured bots and auctions whose **current bidder** is a configured bot. With this config's `BidAgainstPlayers = true`, non-bot player bids can be considered. The query supplies a shared candidate pool, but a house-specific invocation can act only on auctions it resolves in that house. Invalid or wrong-house picks consume attempts too.

Auctions are selected randomly, not by how much they undercut other listings. Small or changing pools, house matching and already-processed auctions affect throughput; there is no universal fixed examinations-per-hour rate.

### 2. Ordinary buying budget

For each valid candidate, the buyer calls the pricing function again. It does not use the seller's previous roll or an addon's market average.

Let **W** be the freshly calculated acceptable unit value and **n** the actual stack count. With the current acceptable-price multiplier of **1.0**, the ordinary stack budget is **W × n**.

Override buyout variations are approximately **−20% / +15%**. Formula-priced items can have additional formula effects and rerolls, so one percentage rule does not describe every runtime item.

### 3. Buyout or bid, followed by the vendor guard

- A buyout must be positive and **strictly less than the ordinary stack budget** to select the buyout branch.
- Otherwise, if there is no current bid, a starting bid **at most the ordinary budget** can be selected. With `AlwaysBidMaxCalculatedPrice = false`, the selected initial amount is the start bid.
- For an existing bid, the current bid plus the required increment must be **strictly below the ordinary budget**.
- After selecting an action, the positive vendor cap can reject it. A selected buyout rejected here **does not fall back to the starting bid during that evaluation**.

The positive guard uses **SellPrice: what an NPC pays you**, not the NPC's retail charge. Its cap applies to the **whole auction**, not to every unit in a stack. An action may equal the cap, but must still pass its separate ordinary-budget test.

The checked direct vendor lookup produces **2,467 positive caps**. It excludes class 7 (Trade Goods); zero-SellPrice cases bypass this check. It is not every NPC's complete stock: event-only or other entries missing from that direct lookup do not gain a positive cap merely because a vendor can sell the item.

Of the 2,467 records, **1,134** are bound and/or timed templates that normally cannot be auctioned. The remaining **1,333** have no identified static binding/duration block in the checked templates, which is not a guarantee of live tradability or availability.

### 4. What a permitted price does—and does not—establish

For a normally auctionable, **uncapped** item, a positive buyout of:

```text
lowest verified buyer unit budget × quantity − 1 copper
```

passes the ordinary buyout test across the checked valuations. Do not multiply a positive vendor cap by stack size. Do not substitute the cheapest visible listing for a verified minimum budget.

**Example:** Iron Riveted War Helm (45107) has a checked minimum unit budget of **4,000g 64s 40c**, with no positive cap in the lookup. A one-item buyout of **4,000g 64s 39c** passes that budget test. The buyer must still be active, select the eligible auction in the correct house, find valid auction/item data and complete the normal transaction. It is not a guaranteed sale before expiry.

Choose a starting bid you are willing to accept. While a configured bot remains the current bidder, that auction is excluded from further buyer consideration; a real-player outbid can change that state. A low start bid is not a harmless placeholder.

### 5. Addons, fees and payment

Compatible auction addons can still scan, compare unit/stack prices and assist posting. Their suggested market prices or undercuts do not guarantee bot acceptance. A custom addon using the correct exported caps and current pricing rules can calculate bounds, but cannot promise selection or the next random valuation.

There is no additional market-demand score or addon approval test in the inspected buyer routine. Seller blacklists, seller quality filters and seller listing weights do not themselves prohibit buyer purchases. The inspected routine also does **not** check or debit a normal player-wallet balance before its direct auction/mail updates.

Auction cuts reduce successful receipts; minimum deposits, rounding and unsold expiry affect costs. Under the documented **5% cut and rate=1 assumption**, a 52c sale nets 50c and a 5c sale nets 5c because the fee truncates. Returned deposit principal is not profit. Actual house/world rates and mail delays may differ. Acquisition cost and lost deposits still matter when assessing profitability.

## Additional Resources

[Advanced Pricing Calculator](tools/AdvancedPricingCalculator/Advanced_Pricing_Calculator.xlsx) — an aid for visualizing advanced **runtime formula** settings. It does not recalculate the 12,906 stored overrides or certify actual auction proceeds.

The documentation's results are source-level/configuration calculations against a pinned database baseline, not a full live-server or addon-compatibility certification. Recheck them if the config, module, core, world data or custom scripts change.

## Credits

- NathanHandley: Created this rewrite of the one that was ported to AzerothCore
- Zeb139: Lots of great enhancements, like enhanced pricing formulas and performance improvements (plus more)
- Ayase: ported the bot to AzerothCore
- Other contributors (check the contributors list)
