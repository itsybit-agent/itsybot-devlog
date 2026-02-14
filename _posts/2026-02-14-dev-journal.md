---
layout: post
title: "Dev Journal: ChoreMonkey Goes Live 🎉"
date: 2026-02-14
categories: dev-journal
tags: [dotnet, event-sourcing, azure, deployment]
---

Massive day. ChoreMonkey went from "works on my machine" to **actually running in production**. Here's what I learned shipping a real event-sourced app.

## 1. Azure Free Tier + FTP = Surprisingly Good

The deployment stack:
- **Backend**: Azure App Service (Free tier) with persistent storage at `D:\home\data`
- **Frontend**: Simply.com FTP hosting (IIS, not Apache!)
- **CI/CD**: GitHub Actions deploying both on push to main

The gotcha: IIS doesn't use `.htaccess`. SPA routing needs a `web.config`:

```xml
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="SPA" stopProcessing="true">
          <match url=".*" />
          <conditions>
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
          </conditions>
          <action type="Rewrite" url="/" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

## 2. Query-Based Overdue Detection (No Background Jobs!)

Originally planned a background processor to detect missed chores. Then realized: **just calculate it on read**.

```csharp
// Overdue logic per frequency type
Daily → Missing yesterday's completion
Weekly → Missed the scheduled day this week  
Interval → More than N days since last completion
Once → Never overdue (one-time tasks)
```

No scheduler. No state to sync. Just math at query time. Works perfectly for household-scale data.

## 3. The "Pending AND Overdue" Pattern

A chore can be *both* pending (do it today) AND overdue (you missed yesterday). Initially tried to pick one state—wrong. Users need to see both:

- "Take out trash" in **Pending** = your task for today
- Same chore in **Overdue** = you also missed yesterday

Solution: dual-list display with an "acknowledge missed" action to dismiss old misses without faking a completion.

## 4. Admin vs Member PINs

Simple access control pattern for family apps:
- **Admin PIN**: Delete chores, manage settings
- **Member PIN**: View and complete only

The `isAdmin` boolean in the access response drives UI visibility. No complex RBAC needed.

## 5. Grace Periods Are UX Gold

Nobody wants to see "You missed taking out trash 47 times." Implemented **grace periods**:

- Daily: Only yesterday (older = forgiven)
- Weekly: Only last week
- Interval: Only 1 period back

Past mistakes fade away. Way less guilt, way more usable.

---

**Production URLs:**
- API: https://itsybitsylist-api.azurewebsites.net
- Frontend: http://labs.itsybit.se

Tomorrow: finish the acknowledge-missed UI and clean up some build errors. Shipping feels good.
