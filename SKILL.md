---
name: linkedin-withdraw-old-page-invitations
description: "Withdraw old LinkedIn page invitations. Use when the user asks to withdraw, remove, or clean up sent LinkedIn page invitations older than 1 month. Navigates to the LinkedIn Invitation Manager, loads all sent page invitations, and withdraws every invitation older than 1 month while keeping recent ones (1 month or newer)."
---

# LinkedIn: Withdraw Old Page Invitations

## Overview

This skill automates withdrawing all sent LinkedIn **page** invitations (not people invitations) that are older than 1 month. It keeps all invitations sent within the last month.

**URL**: `https://www.linkedin.com/mynetwork/invitation-manager/sent/ORGANIZATION/`

---

## Step-by-Step Process

### Step 1 — Navigate to the Page

```
Navigate to: https://www.linkedin.com/mynetwork/invitation-manager/sent/ORGANIZATION/
Wait 3 seconds for the page to load.
```

### Step 2 — Scroll to Load All Invitations

LinkedIn uses infinite scroll. Run this JS to load everything:

```javascript
(function() {
  const main = document.querySelector('main');
  let count = 0;
  const interval = setInterval(() => {
    main.scrollBy(0, 2000);
    window.scrollBy(0, 2000);
    count++;
    if (count >= 600) clearInterval(interval);
  }, 150);
  return 'scrolling started';
})()
```

Wait ~30 seconds for all items to load, then scroll back to top:

```javascript
window.scrollTo(0, 0);
document.querySelector('main').scrollTo(0, 0);
```

### Step 3 — Count Old Items

```javascript
(function() {
  const root = document.getElementById('root');
  const oldTs = ['2 months ago','3 months ago','4 months ago','5 months ago',
    '6 months ago','7 months ago','8 months ago','9 months ago','10 months ago',
    '11 months ago','1 year ago','2 years ago'];
  const allLinks = Array.from(root.querySelectorAll('a[aria-label*="Withdraw invitation sent to"]'));
  let oldCount = 0, firstName = null;
  for (const link of allLinks) {
    let el = link;
    for (let i = 0; i < 5; i++) {
      el = el?.parentElement;
      if (!el) break;
      if (el.textContent.length < 500 && oldTs.some(t => el.textContent.includes(t))) {
        oldCount++;
        if (!firstName) firstName = link.getAttribute('aria-label').replace('Withdraw invitation sent to ','');
        break;
      }
    }
  }
  return { total: allLinks.length, oldCount, firstName };
})()
```

Report the count to the user before proceeding.

### Step 4 — Withdraw Old Invitations (Main Loop)

**Important**: The "Withdraw" button for the first old invitation appears at approximately **x=779, y=499** in the viewport. After scrolling to top, this position is consistent.

Repeat until `oldCount = 0`:

1. **Find & scroll to first old item:**
```javascript
(function() {
  const root = document.getElementById('root');
  const oldTs = ['2 months ago','3 months ago','4 months ago','5 months ago',
    '6 months ago','7 months ago','8 months ago','9 months ago','10 months ago',
    '11 months ago','1 year ago','2 years ago'];
  const allLinks = Array.from(root.querySelectorAll('a[aria-label*="Withdraw invitation sent to"]'));
  for (const link of allLinks) {
    let el = link;
    for (let i = 0; i < 5; i++) {
      el = el?.parentElement;
      if (!el) break;
      if (el.textContent.length < 500 && oldTs.some(t => el.textContent.includes(t))) {
        link.scrollIntoView({ behavior: 'instant', block: 'center' });
        const rect = link.getBoundingClientRect();
        return { cy: Math.round(rect.top + rect.height/2), name: link.getAttribute('aria-label').replace('Withdraw invitation sent to ',''), found: true };
      }
    }
  }
  return { found: false };
})()
```

2. **Click the Withdraw button** at `(779, 499)` — this opens the confirmation modal.

3. **Confirm** by clicking the blue "Withdraw" button in the modal at `(724, 194)`.

4. **Repeat** for the next old item.

**Efficiency tip**: After each confirmation, immediately click withdraw on the next item. The pattern is:
- Click `(779, 499)` → Click `(724, 194)` → repeat

Check count every 10 withdrawals to track progress.

### Step 5 — Verify Completion

```javascript
(function() {
  const root = document.getElementById('root');
  const oldTs = ['2 months ago','3 months ago','4 months ago','5 months ago',
    '6 months ago','7 months ago','8 months ago','9 months ago','10 months ago',
    '11 months ago','1 year ago','2 years ago'];
  const allLinks = Array.from(root.querySelectorAll('a[aria-label*="Withdraw invitation sent to"]'));
  let oldCount = 0;
  for (const link of allLinks) {
    let el = link;
    for (let i = 0; i < 5; i++) {
      el = el?.parentElement;
      if (!el) break;
      if (el.textContent.length < 500 && oldTs.some(t => el.textContent.includes(t))) {
        oldCount++;
        break;
      }
    }
  }
  return { remaining: allLinks.length, oldRemaining: oldCount };
})()
```

Report result to user: how many were withdrawn and how many recent invitations remain.

---

## Key Technical Notes

- **Withdraw links** are `<a>` elements with `aria-label="Withdraw invitation sent to [Name]"` — NOT `<button>` elements.
- **JS `.click()` does NOT work** — LinkedIn uses React synthetic events. Must use actual mouse clicks via the browser computer tool.
- **Modal confirm button** is at approximately **(724, 194)** — this is consistent across sessions.
- **First old item Withdraw button** is at approximately **(779, 499)** after scrolling to top.
- **"1 month ago"** = keep (not old). Only withdraw **"2 months ago"** and older.
- **Multiple page invitations per person**: Some people appear multiple times (invited to different pages). Each entry must be checked individually.
- **Infinite scroll**: The `<main>` element must be scrolled, not just `window`.
- **`find` tool rate limit**: If the `find` tool hits a 429 error (5-hour rate limit), use coordinate-based clicking instead.

---

## Timestamps Considered "Old" (withdraw these)

- 2 months ago, 3 months ago, 4 months ago, 5 months ago
- 6 months ago, 7 months ago, 8 months ago, 9 months ago
- 10 months ago, 11 months ago, 1 year ago, 2 years ago

## Timestamps to KEEP (do not withdraw)

- Sent X minutes/hours ago
- Sent X days ago  
- Sent 1 week ago, 2 weeks ago, 3 weeks ago
- Sent 1 month ago

---

## Example User Triggers

- "Withdraw my old LinkedIn page invitations"
- "Clean up my LinkedIn sent page invitations older than 1 month"
- "Remove all LinkedIn page invites older than a month"
- "Go to my LinkedIn invitation manager and withdraw old page invites"
