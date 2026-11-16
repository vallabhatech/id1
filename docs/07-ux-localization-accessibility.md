# 07 — UX, Localization and Accessibility

## 1. Experience principles

1. **Farmer-first:** Put the farmer's most common actions ahead of administrative features.
2. **Clarity before density:** Present one decision at a time; avoid dashboards filled with unexplained charts.
3. **Voice is optional:** Voice helps reduce typing, but every important action has a visible text/manual path.
4. **Trust through context:** Show market, commodity, variety, unit, date, source and freshness beside prices.
5. **No hidden commitment:** Clearly distinguish saving a draft, publishing a listing, sending a bid and accepting an offer.
6. **Respect local language:** Use farmer-tested terms, not merely literal machine translation.
7. **Design for weak connectivity:** Keep drafts safe and explain what is and is not synced.

## 2. Proposed language rollout

| Stage | Languages | Notes |
|---|---|---|
| Pilot | Telugu, Hindi and English | Proposed initial set; confirm farmer demand and speech support before release |
| Expansion | Additional regional languages based on pilot geography | Add after translation review and user testing |
| All stages | Language preference is user-controlled | Store locale preference independently of canonical price/unit values |

The selected language must not change stored numbers, dates, currency codes, identifiers or units. Display formatting can change while the underlying value remains canonical.

## 3. Information architecture

### Farmer Android app

| Area | Screens | Primary action |
|---|---|---|
| Onboarding | Welcome, language, role, sign-in, profile | Set up an account |
| Home | Market snapshot, quick actions, recent activity | Start a task |
| Market | Crop search, market results, market detail, trend, comparison | Explore prices |
| Forecast | Forecast range, source/data status, explanation | Understand estimate |
| Sell produce | Create lot, add photos, quantity, quality, availability, preview | Publish listing |
| Offers | Listing bid inbox, bid detail, compare offers, accept/decline | Decide on offer |
| Messages | Conversation list, conversation detail | Contact participant |
| Account | Language, profile, help, privacy, sign out | Manage account |

### Buyer web portal

| Area | Screens | Primary action |
|---|---|---|
| Discovery | Search, filters, listing details | Find produce |
| Buyer workspace | Saved searches, own bids, messages | Manage buying |
| Bid flow | Bid form, confirmation, bid history | Submit an offer |
| Account | Business profile, contact preferences, help | Manage account |

### Admin/support portal

| Area | Screens | Primary action |
|---|---|---|
| Operations overview | Feed freshness, failed jobs, reports | Monitor service |
| Reference data | Markets, commodities, units | Maintain controlled references |
| Moderation | Listing/report queue, review detail | Review or resolve |
| Audit | Privileged actions and incident trail | Investigate safely |

## 4. Key farmer flow — check a price

~~~mermaid
flowchart TD
  A[Home] --> B[Select crop or speak crop name]
  B --> C[Confirm crop and variety]
  C --> D[Select district or nearby market]
  D --> E[View observed price range]
  E --> F[View historical trend]
  F --> G{Forecast available?}
  G -- Yes --> H[View estimate and uncertainty range]
  G -- No --> I[Show why estimate is unavailable]
  H --> J[Compare markets or create listing]
  I --> J
~~~

Important UI rules:
- Label the observed modal price separately from a model estimate.
- Show source observation date and retrieval/update time.
- Explain the price unit and allow a supported unit conversion only when dimensionally valid.
- Show a useful empty state when no observation exists.
- Never use a green upward arrow or other visual that implies a guaranteed future gain.

## 5. Key farmer flow — publish produce

1. Farmer chooses the commodity and optional variety.
2. Farmer enters quantity and selects a unit.
3. Farmer adds quality notes and optional photos.
4. Farmer selects availability date and a public location label.
5. App shows a preview and explains what is public.
6. Farmer confirms publication.
7. Listing status and any review state are visible.
8. Farmer can pause, edit or close according to the listing state.

Require explicit confirmation before publishing. Do not publish a partially completed form or a voice-recognized quantity without review.

## 6. Key farmer flow — compare offers

Offer comparison should show:

| Dimension | Why it matters |
|---|---|
| Offered price | Headline offer amount |
| Price unit | Prevents comparing quintal with kilogram without conversion |
| Offered quantity | Partial bids may not cover the whole lot |
| Gross amount | Price multiplied by accepted quantity |
| Known transport/handling costs | Allows an estimated net comparison |
| Quality and weighing conditions | Shows conditions that could change final settlement |
| Pickup/delivery terms | Clarifies responsibility |
| Bid validity | Shows when the offer expires |
| Buyer status | Displays only the verification checks actually completed |

The comparison must separate a known total from an estimate. If transport cost is unknown, do not display a precise net amount.

## 7. Voice interaction design

### Supported intent families for an initial voice assistant

- Ask for current observed price for a crop/market.
- Ask for recent price history.
- Search for nearby supported markets.
- Start a new produce listing.
- Read bid status and offer details for the signed-in farmer.
- Open a help explanation.

### Safe voice flow

1. User presses a clearly labelled microphone control.
2. App explains microphone use and requests permission only when needed.
3. Spoken input is converted into text by the configured device/service speech capability.
4. App displays recognized text.
5. Extracted crop, market, quantity and unit are shown for confirmation.
6. Consequential actions require a visible confirmation step.
7. User can edit the text or switch to manual input.

Do not treat speech recognition as identity verification or consent to a sale.

### Speech output

- Speak short, structured responses.
- Read price, currency and unit explicitly.
- Repeat date and market name when ambiguity is possible.
- Offer a replay control.
- Avoid reading long legal or privacy text aloud as the only disclosure.

## 8. Localization workflow

- Maintain translation keys in resource files; do not hard-code UI text.
- Keep glossary entries for crop names, varieties, market terms, weights, bid states and common farming language.
- Use native reviewers for important farmer-facing text.
- Test all numeric and date formats in each locale.
- Provide a fallback language when a translation is missing.
- Never translate source IDs or silently change the meaning of a market grade or price unit.
- Record the translation version used for critical consent text.

## 9. Accessibility and inclusive design checklist

- Touch targets large enough for comfortable use.
- Readable contrast in daylight and low-brightness conditions.
- Text can scale without clipping or hiding actions.
- Screen-reader labels describe the actual purpose of controls.
- Icons are accompanied by text labels for critical actions.
- Color is never the only way to show bid or listing status.
- Error messages explain the fix in the selected language.
- Form fields have labels, units and examples.
- Voice input has a keyboard/manual alternative.
- Loading, offline, retry and sync status are announced accessibly.
- Avoid time-critical interactions that depend on fast reading.

## 10. Low-connectivity behavior

- Cache reference data and recent search results with a visible last-updated time.
- Save incomplete listing forms locally and mark them as unsynced.
- Do not claim that a bid or message was sent until the server confirms it.
- Use bounded retry with idempotency for mutations.
- Show a clear conflict message if a listing changed on another device.
- Provide a manual refresh action.
- Do not cache sensitive messages or exact locations beyond the defined security policy.

## 11. Usability validation plan

Test with farmers and buyers in the pilot geography. Include different ages, device capabilities, levels of digital confidence and language preferences. Ask participants to complete realistic tasks rather than rate mockups only.

Suggested tasks:
- Find a crop's observed price in a selected market.
- Explain the difference between modal price and a forecast.
- Create and preview a listing.
- Compare two offers with different units or quantities.
- Find a conversation and report a suspicious listing.
- Complete the same key task without voice.

Record completion, errors, confusion points and participant language. Do not collect personal farm or financial information that is not needed for the session.
