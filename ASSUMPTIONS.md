# Assumptions

The following explains some core concepts regarding the LoansBot services in
no particular order. Most of these assumptions are not surprising, but they
still need to be taken into account in how the infrastructure is set up:

- The website and the LoansBot share the same reddit API restrictions. That is
  to say - neither the website nor the LoansBot can know (without
  communication) how much API time is available, and it is in bad faith to
  force reddit to ratelimit rather than self-ratelimiting.
- The website can influence how the LoansBot operates in temporary ways. The
  most direct example is rechecks - the website tells the LoansBot to look at
  a comment it has already seen as if it had not seen it before.
- The website can influence how the LoansBot operates in permanent ways. The
  most direct approach is responses - the responses that the LoansBot makes
  are formatted, and that format can be altered via the website.
- Both the LoansBot and the website may make authoritative changes to the core
  database - the list of loans.
- Users are granted certain permissions based on their state on reddit, which
  our application does not have authoritative control over. For example, users
  which are below a certain amount of karma will not have their comments
  processed by the LoansBot. Alternatively, users are granted certain
  permissions if they are listed as an approved submitter of r/Borrow. These
  permissions can change at any time, and should be captured by the LoansBot
  reasonably quickly.
- Users are granted certain permissions based on computed values within our
  database - such as the number of completed loans. The main complexity here
  is that, again, neither the website nor the LoansBot exclusively control this
  state.
- Users may be granted certain permissions explicitly via the website.
- Users may have certain permissions explicitly revoked via the website.
- Users may be granted certain permissions explicitly via commands to the
  loansbot.
- Users may have certain permissions explicitly revoked via commands to the
  loansbot.
- Loans are made in many currencies, yet we still need a way to aggregate them
  under a particular currency. Similarly, exchange rates varies over time, so
  a given loan can be considered completely repaid even when the USD-equivalent
  at repayment time is below the USD-equivalent at loan time.
- It is not free to convert currencies.
- Reddit can and does go down, and during this time as many operations from the
  website as possible should function as normally. Furthermore, once Reddit
  recovers the LoansBot should resume normal operations "soon" after.
- Reddit accounts can get deleted or shadow-banned at any time. That is, when
  we see a comment by a user, we are not guarranteed that the user won't be
  deleted before we are able to view their account page.
