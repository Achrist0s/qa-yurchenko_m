\# BUG-01: Navbar cart counter does not update after removing all items from the cart, without a page reload



\---



\*\*Environment:\*\* prod https://school-web-ncqi.onrender.com/catalogue 



\*\*Preconditions:\*\* User has at least 1 item in the cart; navbar shows "Cart (N)" with N > 0.



\*\*Steps to reproduce:\*\*



1\. Open the Cart page https://school-web-ncqi.onrender.com/cart.

2\. Remove all items from the cart.

3\. Without reloading the page, look at the "Cart (N)" counter in the navbar.



\*\*Actual result:\*\* The Cart page correctly shows "Your cart is empty", but the navbar counter still shows the stale value (e.g. "Cart (1)"). It only updates to the correct value after a manual page reload (F5).



\*\*Expected result:\*\* The navbar cart counter should update immediately after items are removed, without requiring a page reload — the navbar should stay in sync with the actual cart state in real time. \*Source: basic expected UI behavior — a global element should reflect the current app state; not a documented requirement, this is the direct contradiction between two simultaneously visible states (empty cart page vs. non-zero navbar counter).\*



\*\*Severity:\*\* Major — the app displays two contradictory states of the same cart at the same time, which is a state-consistency defect, not cosmetic. 



\*\*Priority:\*\* Medium — the underlying data is correct (cart is actually empty) and the issue self-resolves on reload, so it doesn't block any flow, but it undermines user trust in the UI and should be fixed soon.



Attachments: https://monosnap.ai/file/CR5BsVsSiUyX2v4FYu0TwFZulxmryb 



\---



\## BUG-02: Adding the same product multiple times creates separate cart cards instead of increasing quantity



\*\*Environment:\*\* prod https://school-web-ncqi.onrender.com/catalogue 



\*\*Preconditions:\*\* Cart is empty at the start of the scenario.



\*\*Steps to reproduce:\*\*



1\. Open the Catalogue page https://school-web-ncqi.onrender.com/catalogue.

2\. Find any product and click "Add to cart".

3\. Without navigating away, click "Add to cart" on the same product again.

4\. Open the Cart page https://school-web-ncqi.onrender.com/cart.



\*\*Actual result:\*\* Two separate cards for the same product appear in the cart, each with quantity 1, instead of one card with quantity 2.



\*\*Expected result:\*\* Adding the same product again should result in a single cart entry with an incremented quantity (2), not two separate rows. \*Source: hypothesis — standard e-commerce cart behavior (product unique by ID, quantity incremented); requires confirmation from PO whether this app's spec defines the same logic.\*



\*\*Severity:\*\* Major — a logic defect in a core flow (cart) that can directly affect order correctness, e.g. if duplicates are later billed as two separate full-price line items instead of one line at quantity 2. 



\*\*Priority:\*\* High — adding the same item twice is a common, everyday user action, so this affects a large share of users.



Attachments: https://monosnap.ai/file/NClnQAg1HUHKPOHYzeJWKafhkZURiN



\---



\## BUG-03: PATCH /users/me accepts an invalid phone number (letters mixed with digits) and returns 200 OK



\*\*Environment:\*\* prod https://school-web-ncqi.onrender.com/profile



\*\*Preconditions:\*\* User is logged in and on the Profile page with an editable "Phone" field.



\*\*Steps to reproduce:\*\*



1\. Open the Profile page. https://school-web-ncqi.onrender.com/profile

2\. In the "Phone" field, enter `56467567рпсапчпчапспррп` (digits mixed with Cyrillic letters).

3\. Save the form.



\*\*Actual result:\*\* The request `PATCH https://school-api-vqyw.onrender.com/users/me` returns \*\*200 OK\*\* — the invalid value is accepted and persisted with no validation error.



\*\*Expected result:\*\* The server should reject the request with \*\*400 Bad Request\*\* and a validation message; the "Phone" field should accept digits only (optionally `+`, spaces, parentheses depending on the required format), not letters. \*Source: hypothesis — a phone number containing letters is not a phone number by definition; the exact accepted format/regex requires confirmation from PO/spec.\*



\*\*Severity:\*\* Major — invalid data is persisted to the database, corrupting data integrity and potentially breaking any feature that later depends on a valid phone number (e.g. SMS order confirmation, support contact). 



\*\*Priority:\*\* High — affects the integrity of every user's personal data on profile edit; the longer this remains unfixed, the more corrupted records accumulate.



Attachments: https://monosnap.ai/file/ajZRe5BkU3Y26dyhS7to9FbrFzvCWZ



\---



\## BUG-04: Price sorting does not work in either direction ("high to low" nor "low to high") — Catalogue



\*\*Environment:\*\* prod https://school-web-ncqi.onrender.com/catalogue



\*\*Preconditions:\*\* User is on the Catalogue page with "All categories" selected (no active price/search filters).



\*\*Steps to reproduce:\*\*



1\. Open the Catalogue page. https://school-web-ncqi.onrender.com/catalogue

2\. Select "Price: high to low" in "Sort by"; note the prices of the first few products.

3\. Select "Price: low to high"; note the prices of the first few products.



\*\*Actual result:\*\* In both cases, the product order does not match the selected sort. With "high to low", the observed sequence was $9.99 → $99.00 → $8.99 (not descending). With "low to high", the order was likewise confirmed to not be ascending.



\*\*Expected result:\*\* "Price: high to low" should return products in descending price order; "Price: low to high" should return products in ascending price order. \*Source: the sort option labels themselves — this is a direct functional requirement, not a hypothesis.\*



\*\*Severity:\*\* Major — a core, explicitly labeled catalog function (price sorting) fails to do what it claims, in both directions. 



\*\*Priority:\*\* High — price sorting is a standard, frequently used navigation tool; the break affects every user who relies on it, regardless of direction chosen.



Attachments: https://monosnap.ai/file/3xGrlkCxY5woAdXeZabJxjJk6Q7h6t



\---



\## BUG-05: Invalid request with minPrice=-1 is sent to the backend and returns 400, instead of being blocked client-side — Catalogue



\*\*Environment:\*\* prod https://school-web-ncqi.onrender.com/catalogue



\*\*Preconditions:\*\* User is on the Catalogue page with default filters.



\*\*Steps to reproduce:\*\*



1\. Open the Catalogue page https://school-web-ncqi.onrender.com/catalogue, with DevTools → Network (Fetch/XHR filter) open.

2\. In the "Lowest price (cents)" field, enter -`1`.

3\. Observe the Network tab and the on-screen message.



\*\*Actual result:\*\* A request `GET /products?minPrice=-1\&sort=name\_asc\&page=1` is sent and returns \*\*400 Bad Request\*\*. Only after this response does the UI show a generic "Check the highlighted fields" message with the field highlighted — the client does not prevent the invalid value from being submitted; it relies on the backend's rejection.



\*\*Expected result:\*\* The "Lowest price" field should be validated client-side before the request is sent (negative values are invalid), avoiding an unnecessary API call; the error message should state the specific reason (e.g. "must be 0 or greater") rather than a generic text. \*Source: hypothesis — standard client-side form validation practice; the exact message wording requires confirmation from PO/design.\*



\*\*Severity:\*\* Minor — the backend correctly rejects the invalid data; there is no broken functionality or data leak. 



\*\*Priority:\*\* Low — does not block the purchase flow, only causes an extra network call and an unclear message; can be fixed in the normal backlog order.



Attachments: https://monosnap.ai/file/wFEAfbNiqUQP3FSk5cKGDjX73VUChc



\---



\## BUG-06: Products with a corrupted "A" character (Å/Ä) are not found via search using their original name — Catalogue



\*\*Environment:\*\* prod https://school-web-ncqi.onrender.com/catalogue



\*\*Preconditions:\*\* Products "Apple crate" and "Armchair" exist in the catalog, displaying with a corrupted first character ("Åpple crate", "Ärmchair" — see BUG-07).



\*\*Steps to reproduce:\*\*



1\. Open the Catalogue page. https://school-web-ncqi.onrender.com/catalogue

2\. Enter "Armchair" (plain Latin "A") into the search field and submit.



\*\*Actual result:\*\* Neither search returns the corresponding product — the result is empty / "no results found", even though the product is present in the catalog (visible when browsing without a filter).



\*\*Expected result:\*\* Searching for "Apple crate" and "Armchair" should return the matching product. \*Source: basic expected search behavior — user searches using the name they believe is correct (standard English spelling, no diacritics); requires confirmation from PO on which spelling is considered canonical for search indexing.\*



\*\*Severity:\*\* Major — this is not cosmetic: a user searching with the correct spelling gets a false "no results" and cannot find a product that actually exists — a direct loss of core search functionality. 



\*\*Priority:\*\* High — affects any user trying to find these products the normal way, and shares a likely root cause with BUG-07, so both should be fixed together.



Attachments: https://monosnap.ai/file/qYsxaijIwUinq8vRyvUQ6reXIjqgeE



\---



\## BUG-07: Product names starting with "A" display a corrupted first character (encoding issue) — Catalogue



\*\*Environment:\*\* prod https://school-web-ncqi.onrender.com/catalogue



\*\*Preconditions:\*\* At least two products whose names originally start with "A" exist in the catalog: "Apple crate" and "Armchair".



\*\*Steps to reproduce:\*\*



1\. Open the Catalogue page. https://school-web-ncqi.onrender.com/catalogue

2\. Find the "Apple crate" product card.

3\. Find the "Armchair" product card.

4\. Compare the first character of the name and description on both cards.



\*\*Actual result:\*\* Both products show a corrupted first character in both name and description: "\*\*Å\*\*pple crate" (instead of "Apple crate") and "\*\*Ä\*\*rmchair" (instead of "Armchair"). The corruption character differs (Å vs. Ä), but the pattern is consistent — Latin "A" replaced with an "A" carrying a diacritic mark.



\*\*Expected result:\*\* Product names should display without diacritics: "Apple crate" and "Armchair", consistent with the rest of the English text. \*Source: hypothesis — standard English product naming has no such diacritic; requires confirmation from PO and, more importantly, verification of the data source (likely an encoding error during import/generation, not a manual typo).\*



\*\*Severity:\*\* Minor — a text rendering issue that does not by itself break functionality (search, cart, checkout still work). 



\*\*Priority:\*\* High — the fact that the pattern repeats across different products with different diacritics indicates a systemic encoding bug likely affecting \*\*every\*\* product starting with "A" (and potentially other letters), so the root cause should be fixed rather than patched name by name.



Attachments: https://monosnap.ai/file/RoYxkraSLv5I9pbZSpZskbXCuH0n5M



\---



\## BUG-08: Cart items are not cleared after logging out



\*\*Environment:\*\* prod https://school-web-ncqi.onrender.com/catalogue



\*\*Preconditions:\*\* User is logged in; cart is empty at the start of the scenario.



\*\*Steps to reproduce:\*\*



1\. Log in to an account.

2\. Add 1–2 products to the cart.

3\. Verify the items appear in the Cart.

4\. Click "Logout".

5\. Check the cart contents (navbar counter) without logging back in.



\*\*Actual result:\*\* After logging out, the items added during the logged-in session are still present in the cart, as if added by an anonymous guest.



\*\*Expected result:\*\* After logout, the cart tied to that user should be cleared (or at least not shown to an anonymous visitor) — items belonging to one account should not remain visible after logout. \*Source: hypothesis — basic expected separation of state between sessions/users; the exact intended business logic (whether a guest should have their own separate cart at all) requires confirmation from PO/spec.\*



\*\*Severity:\*\* Major — this mixes data across authorization states: an anonymous visitor sees a cart populated by a logged-in account, which is a logic defect, not cosmetic. 



\*\*Priority:\*\* High — on a shared/public device this also becomes a privacy concern (the next user of the browser sees what the previous user was buying), and "add item → log out" is a common everyday action.



Attachments: https://monosnap.ai/file/H2lFZksVyXCzWfqWbvea5Hh7dJnfdn



\---



\## BUG-09: Address fields (label, line1, city, postalCode) accept arbitrary text with no format validation — Addresses in Profile



\*\*Environment: prod\*\* https://school-web-ncqi.onrender.com/profile



\*\*Preconditions:\*\* User is logged in and on the Addresses section of the Profile page.



\*\*Steps to reproduce:\*\*



1\. Open the Addresses section in Profile. https://school-web-ncqi.onrender.com/profile

2\. Fill the label, line1, city, and postalCode fields with ' OR '1'='1 ; fill "country" with a valid value.

3\. Save the address.



\*\*Actual result:\*\* `POST /users/me/addresses` succeeds with no error. `GET /users/me/addresses` confirms the record was persisted: `label: ' OR '1'='1`, `line1:' OR '1'='1`, `city: ' OR '1'='1`, `postalCode: ' OR '1'='1` — every text field accepted content that is clearly not valid address data; the record is even flagged `isDefault: true`.



\*\*Expected result:\*\* Address fields should have basic format validation — e.g. "city" should not accept a string containing quotes and logical operators, "postalCode" should match a postal-code format (digits/specific mask). The server should reject such values with 400 and a message specifying which field failed and why. \*Source: hypothesis — basic input validation for semantically defined fields (address, postal code); the exact required format/regex per field requires confirmation from PO/spec.\*



\*\*Note for the assignee:\*\* the SQL-injection-style payload did \*\*not\*\* execute as an attack — it was stored as a plain string, with no sign of query execution or data leak. This should be stated explicitly so the assignee focuses on the missing format validation, not a non-existent injection vulnerability.



\*\*Severity:\*\* Major — the address technically "works" (saves and reads back correctly), but the total lack of validation lets arbitrary/malicious-looking content pollute the database, which can break downstream consumers of this data (e.g. printing a shipping label, passing the address to a courier integration). 



\*\*Priority:\*\* Medium — does not block any current critical business flow and the injection attempt did not succeed, so this is a data-quality gap rather than a critical security issue, to be addressed in normal priority order.



Attachments: https://monosnap.ai/file/zUVhT4N3qsaNc0j4TsDD6dYsj27Zgj



\---



\## BUG-10: "Country" field accepts any two characters, including non-letters, instead of a valid two-letter country code — Addresses in Profile



\*\*Environment:\*\* \*\*prod\*\* https://school-web-ncqi.onrender.com/profile



\*\*Preconditions:\*\* User is logged in and on the Addresses section of the Profile page.



\*\*Steps to reproduce:\*\*



1\. Open the Addresses section in Profile.

2\. In the "Country" field, enter "Г\*" (one non-Latin letter plus a non-letter character, 2 characters total).

3\. Save the address.



\*\*Actual result:\*\* The value "Г\*" is accepted with no validation error and persisted as the country value, despite: (a) "\*" not being a letter at all, and (b) "Г" not belonging to the Latin alphabet used by country codes.



\*\*Expected result:\*\* The "Country" field should only accept a valid two-letter country code per ISO 3166-1 alpha-2 — two \*\*Latin\*\* letters from the set of real, existing country codes (e.g. "UA", "US", "PL") — not any two arbitrary characters. A value like "Г\*" should be rejected with an explanatory error. \*Source: the field's own hint text, "Use a two-letter country code" — this is a direct contradiction of the field's stated requirement, not a hypothesis.\*



\*\*Severity:\*\* Major — the field's validation does not enforce its own stated requirement in any respect: it accepts a non-letter character and a letter outside the country-code alphabet, corrupting data quality (an address with an invalid country code is unusable for real shipping/logistics integrations). 



\*\*Priority:\*\* Medium — does not block saving an address right now, and is less severe than storing malicious content (see BUG-09), but directly affects the quality of every address saved through this form.



Attachments: https://monosnap.ai/file/LixclEvQ0GNYimDApKHXRTaedw2pb0

