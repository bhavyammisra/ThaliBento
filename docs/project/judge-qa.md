# Judge Q&A sheet (final pitch, 25 Oct)

One line to say out loud, then the proof if they push. Links are the public sources we quoted; paths like `03-research/…` point to the team's private research notes.

## 1. "Doesn't the booking system already carry the meal over on a rebooking?"
**Say:** Sometimes, inside one airline. Across a codeshare, or after a reissue, it often doesn't, and nobody is told. We only raise an alert when the meal code is actually missing from the next passenger list, so when it does carry over, nobody sees anything.
- Delta's own rules: a meal requested through another carrier "will not be applied", must be re-requested after voluntary changes, and isn't guaranteed after a disruption rebooking within 24 h ([Delta Pro](https://pro.delta.com/content/agency/hk/en/products-and-services/in-flight/special-meal-requests--spml-.html)).
- JAL's IndiGo codeshare feeds about 15 domestic cities each into Delhi and Bengaluru ([JAL 008473](https://press.jal.co.jp/en/release/202411/008473.html)); 6E codes on JAL India flights since Feb 2026 ([AeroRoutes](https://www.aeroroutes.com/eng/260122-6ejlcodeshare)).
- Demo proof: Act 2. The 23:02 ADL drops Priya's AVML; the alert shows the raw before/after lines and 1 h 38 m to the caterer's list.

## 2. "Who pays for the backups?"
**Say:** JAL, the same way it pays for caterer spares today. They're sealed, shelf-stable veg meals, so an unused one flies again on the next flight instead of being thrown away. The pilot uses 4 per flight; after month 1, our forecast sizes the stock from real late requests and festival dates.
- Retort-packed Indian meals keep about 12 months at room temperature (DRDO/CFTRI), so there's no cold chain and no daily write-off (`03-research/deep-dive/notes/food-supply.md`).
- The other side of the ledger: Indian courts have awarded ₹40,000 to ₹65,000 for wrong-meal cases (`05-solution/impact.md`), plus the viral-incident cost.
- Unproven: the exact cost per backup. Say "to be priced with the caterer in the pilot" (assumption).

## 3. "JAL already has Torizen veg meals. Why do you need this?"
**Say:** Torizen fixed the recipe. We make sure that recipe reaches the right seat. Torizen covers Japan-departing flights; the gaps are the India-departing legs, Jain meals, IndiGo feeder passengers and last-minute changes.
- Torizen-supervised veg main in Economy on Narita→Delhi/Bengaluru from 1 Jun 2026; AVML in all cabins on Japan-departing flights from 1 Sep 2026 ([JAL 009701](https://press.jal.co.jp/en/release/202609/009701.html)).
- VJML (Jain) isn't part of the Torizen launch; DEL/BLR→Tokyo meals are catered in India (`03-research/deep-dive/report.md`).

## 4. "How is this different from i-PAXINFO?"
**Say:** It isn't a new crew app. i-PAXINFO already shows crew the passengers and meal orders, offline. We add one screen to it: scan the tray, check it against that passenger's food rules, green or red. That check at the moment of serving is the part nobody does today.
- i-PAXINFO: JAL's crew iPad mini app, offline device-to-device sync via Ditto ([Ditto/JAL](https://www.ditto.com/customers/japan-airlines)).
- A tray-against-rules scan at the seat wasn't found at any airline (`03-research/reality-check/existing-practice-2.md`).
- Our crew tablet already runs in airplane mode in the demo (Act 5), so it fits i-PAXINFO's offline model.

## 5. "Privacy: APPI and DPDP?"
**Say:** We store food rules, not religion: "no onion, no garlic", never "Jain". The only religious word that travels is JAL's own meal code (VJML is "Vegetarian Jain meal"), because that's what the airline orders. Consent is asked per purpose, the profile is deleted after the trip, and the tray QR is an opaque id with no personal data.
- APPI treats creed and medical history as "special care-required" data needing prior consent ([Securiti](https://securiti.ai/japan-act-on-the-protection-of-personal-information-appi)).
- DPDP rules require specific, unbundled consent from 13 May 2027 ([Hogan Lovells](https://www.hlc.com/en/publications/indias-digital-personal-data-protection-act-2023-brought-into-force-)).
- In the build: the profile stores only forbid/allow ingredient tags (the quick pick a passenger tapped, like "Jain", stays on their phone), the audit trail and tray label say what's excluded ("VEG · no onion family, no root vegetables…"), the profile has separate consents (this trip, save profile, share with caterer, cross-border) and an expiry date, and we never infer a diet from a name.

## 6. "What if the model misreads a recipe?"
**Say:** The model only reads. Our rules decide, and a person approves every sheet before it becomes a meal option. Anything unclear comes back as "needs a human", never as "safe".
- A Japanese dictionary pass runs first (出汁 dashi → bonito fish), then the model fills a fixed ingredient list with a confidence and the exact text it read.
- The rules engine gives SAFE, UNSAFE or NEEDS-HUMAN and names the rule that fired. Sheets stay "needs review" until approved.
- **Our numbers** (23 hand-labelled hard sheets, vegetarian rules; a stress set, not a random sample): the dictionary finds 79% of hidden non-veg ingredients (19 of 24), the model 96% (23 of 24) with no false alarms, both together 100%. A person clears the dictionary's 5 false alarms before approving. Shown live on the Accuracy tab of `/caterer/recipes` (source: `GET /api/recipes/eval`, m9).
- The demo runs from cached results, so it works with the Wi-Fi off.

## Likely follow-ups
- **"Do you write into Altéa?"** No. We read copies of the PNL/ADL that already go to the airport and the caterer. "Request again" drafts the SSR line; an agent keys it in.
- **"Can the server and the tablet disagree?"** No: the same rules file runs on both, and 625 shared test cases give identical answers in Python and TypeScript.
- **"Why not just pick AVML for every Indian vegetarian?"** AVML is right for many, but "vegetarian" isn't one diet. JAL's AVML is a spicy vegetarian meal with dairy that says nothing about onion or garlic, and its sample menu has potato and carrots, so a Jain needs VJML and someone who avoids onion and garlic needs AVML plus a kitchen note. Our form asks what you don't eat and picks the code and the note.
- **"Isn't HNML clearly labelled non-vegetarian?"** Yes: JAL's English page calls it "Non-Vegetarian Hindu Meal", and the Japanese page says it isn't for vegetarians. The booking itself carries only the code, so when an agent books HNML for a vegetarian, nothing compares it with the passenger. We do: that's 24A in the demo, flagged with the fix (AVML).
