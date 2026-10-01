# Reflection

Answers to the reflection questions from the [project brief](PROJECT-BRIEF.md).

## 1. What assumption in the original documentation was most dangerous?

"Employees never see or handle full card numbers." It was the most dangerous because everyone believed it, so nobody looked. That one sentence hid the biggest part of the real card environment: AR laptops, the VPN, the email system, and the phone vendor's recordings. The other claims were wrong too, but this one kept people from even asking the question.

## 2. If you were the QSA, what would be your first question?

"Walk me through a phone payment, from start to finish." Following one real payment exposes the gap between the diagram and reality faster than any document review. In this case it leads straight to the recordings, the clipboard, the home laptop, and the VPN, none of which appear in the design document.

## 3. What's the single change that would most reduce scope?

Secure payment links. If customers enter their card details themselves, staff never hear, read, or type a card number. That one change takes AR laptops, the VPN, the email system, and the call recordings out of scope at the same time, and it fixes three of my findings at once.

## 4. Where did you have to make a judgment call without clear guidance?

Cleaning up the emails and call recordings that contain card numbers. PCI DSS says to delete card data once it is no longer needed, and never to keep security codes. But the org's retention rules, or a litigation hold, may require keeping those same emails. No rule settles that conflict. My call was that Risk & Compliance should not purge anything on its own: Legal has to define what can be deleted, and anything under a hold should be pulled out of the open inbox and locked down instead of left where anyone can read it.

## 5. What residual risk remains even with perfect implementation of your recommendations?

Two things. First, people. Customers will still email card numbers even with an automatic reply, and staff can still slip up, like writing down a number heard on a call. Training and procedures reduce human error but never remove it. Second, InvoiceTrack. Even once it is cut off from the card environment and out of PCI scope, it is still old, undocumented, and reachable by outside partners. Scope can fix a compliance problem; it cannot fix a fragile system. That risk stays with the business, and it is already tracked in my risk acceptance project.
