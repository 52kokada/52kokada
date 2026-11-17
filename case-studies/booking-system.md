# Case study: booking and payments for a Japanese SME

The client is not named, and no client data is shown.

| | |
|---|---|
| Client | A fairly large Japanese SME that sells bookable classes |
| Before | A WordPress site built in 2013 |
| Delivered | Public site, booking and payment system, staff dashboard |
| Timeline | 26 days from first commit to live card payments, in late 2026 |
| Team | Two people, working with AI coding agents |
| Stack | Cloudflare Workers, D1, R2, Square, Resend, Google Calendar and Sheets |

## Challenge

The client's site was a WordPress installation from 2013. It could not express the booking
rules: classes with a minimum number of guests, group discounts, private classes, gift
certificates, notes for each guest, and refunds that depend on how late a guest cancels.

The rules were collected as 22 written questions to the client. The answers became the
specification for the booking logic.

## Solution

Guests book in English or Japanese. They choose a class, enter each guest's name and notes,
and pay by card.

Staff use a dashboard for the day's classes, bookings, customers, gift certificates, refunds
and message templates. A "needs attention" list shows classes under their minimum, messages
that were not delivered and syncs that failed.

The system is one Cloudflare Worker with a D1 database. Square takes payments, Resend sends
mail, and bookings sync to the client's Google Calendar and Google Sheets.

## Engineering

- Capacity is enforced in the database. Three triggers on the reservation and session tables
  reject a write that would put a class over capacity, so two guests paying at the same time
  cannot both take the last seat.
- All changes go through 16 named actions (book, move, cancel, refund and others). The
  dashboard, the booking page and the payment webhook call the same actions, and each action
  writes a log entry.
- The payment webhook verifies Square's signature. Recording the same payment twice has no
  second effect.
- A second Worker and database run against Square's sandbox. A production release can be made
  only from the main branch, by one script.
- 107 test files cover the rules, the payment paths and the browser flows.

## Incident after launch

Three days after launch, some card payments were declined, and the page said only that payment
was incomplete. I read the causes from Square's webhook logs. The fix shows the guest a plain
reason (expired card, insufficient funds, wrong details) without card digits or raw provider
codes, stops a late event from overwriting a newer result, and releases an unpaid seat after
15 minutes instead of 30.

## Outcome

The system went live 26 days after the first commit and has taken card payments in production
since then. Booking and revenue figures belong to the client and are not published here.

## My role

I wrote the core, the rules engine, the booking actions, the guest booking page and the test
suites, and 135 of the 212 commits to the backend. The dashboard was shared work.
