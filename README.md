# Personal Accountant

A personal mobile expense tracker (P-Accountant) that syncs to a Supabase database.

---

## Setup

### Step 1 — Create a Supabase Integration

1. Go to [Supabase Dashboard](https://supabase.com/dashboard/organizations) and open your workspace.
2. Click your organization name or create a new organization.
3. Create a new project
4. On the Project main page, you will be able to see ur project name, under it is ur **Supabase - Project URL**
5. Click the dropdown beside the url, you will be able to find **Supabase - Publishable Key** 

---

### Step 2 — Create the Expenses Table

1. On the left navigation, redirect to **SQL Editor**
2. **Paste** the following SQL query and run it to create the **Expenses Table**
   ```
   #SQL Query

   create table expenses (
      id uuid primary key default gen_random_uuid(),
   
      name text not null,                -- Title
      amount numeric not null,          -- Amount
      category text,                    -- Select (stored as text)
      date date not null,               -- Date
      notes text,                       -- Rich text
  
      created_at timestamp with time zone default now(),
      updated_at timestamp with time zone default now()
   );
   ```

####[Optional - Low Security]

1. On the left navigation, redirect to **Table Editor**
2. Enable "RLS policies" 
3. On the left navigation, redirect to **Authentication > Policies > Create Policy > Enable Read Access for all Users**
4. On the left navigation, redirect to **SQL Editor**
5. **Paste** the following SQL query and run it to create the **Insert Policy**
   ```
   #SQL Query
   
   alter policy "Enable insert for anon and authenticated"
   on "public"."expenses"
   to public
   with check (
     (auth.role() = ANY (ARRAY['anon'::text, 'authenticated'::text]))
   );
   
   ```
---

### Step 3 — Configure the App

Open the app in your browser. On first launch you'll see a setup screen with these fields:

| Field                        | Value                                              |
|------------------------------|----------------------------------------------------|
| **SUPABASE URL**             | Your Project URL (https://xxxxxxx.supabase.co)     |
| **SUPABASE PUBLISHABLE KEY** | Public key (sb_publishable_xxxxxxxxx)              |
| **Currency Symbol**          | e.g. `$`, `£`, `€`, `S$`                           |
| **Your Name**                | Used for the greeting (optional)                   |

---

## Gmail Auto-Sync (Transaction Alerts)

Transactions are logged automatically by a Google Apps Script that monitors your Gmail bank alert emails. Every 12 hours it fetches any new alerts, parses the amount and merchant, and creates a Supabase Row directly — no manual input needed. The P-Accountant app picks them up on its next sync.

The script calls the Supabase directly and applies a `spendly-processed` Gmail label to every thread (Email) it handles so it never double-processes.

### Step 4 — Set Up the Apps Script

1. Go to [script.google.com](https://script.google.com) → **New project**.
2. Delete the default `myFunction` code and paste the entire contents of [`Gmail-Supabase.gs`](https://github.com/cleontay/P-Accountant/blob/main/Gmail-Supabase.gs).
3. In the editor, open the `setConfig` function, replace the placeholder values with your real credentials, then **run it once** (▶ button). Google will ask you to authorise Gmail and URL Fetch access — approve both. **Ensure that you are running the correct function "setConfig", by checking on the right of (▶ button).

```js
function setConfig() {
  const props = PropertiesService.getScriptProperties();
  props.setProperties({
    SUPABASE_URL: 'https://XXXXXXXXX.supabase.co',
    SUPABASE_KEY: 'sb_publishable_XXXXXXXX',
    SUPABASE_TABLE: 'expenses', // your table name
  });
  console.log('Config saved.');
}
```

4. Run `createTrigger()` once. This registers a time-based trigger that runs `processEmailAlerts` every 12 hours in the background — even when your phone is off. It also saves today's date as the cutoff, so historical emails are permanently ignored.

5. To test immediately, run `processEmailAlerts()` manually and check the **Execution log** (View → Executions).

### Supported Banks

| Bank | Sender | Subject filter |
|------|--------|----------------|
| DBS PayLah! | `paylah.alert@dbs.com` | `Transaction Alerts` |
| DBS Card | `ibanking.alert@dbs.com` | `Card Transaction Alert` |
| Citibank | `alerts@citibank.com.sg` | `Citi Alerts - Credit Card/Ready Credit Transaction` |
| OCBC | `notifications@ocbc.com` | `PayNow transfer made` |
| UOB | `unialerts@uobgroup.com` | `UOB - Transaction Alert` |
| UOB | `unialerts@uobgroup.com` | `UOB - NETS QR payment made` |


### Category Auto-Detection

The script writes the full display name into the Supabase `Category` select field. If no rule matches, it defaults to `Miscellaneous`.

| Supabase cat      | Matched keywords (examples)                              |
|-------------------|----------------------------------------------------------|
| `Transport`       | Grab, taxi, MRT, SBS, EZ-Link                            |
| `Food & Drinks`   | restaurant, cafe, kopitiam, hawker, Starbucks, KFC       |
| `Shopping`        | NTUC, FairPrice, Cold Storage, Sheng Siong, Lazada       |
| `Health`          | clinic, pharmacy, Guardian, Watsons, hospital            |
| `Entertainment`   | cinema, Shaw, Golden Village, museum                     |
| `Subscriptions`   | Netflix, Spotify, Apple, Google Play                     |
| `Travel`          | airlines, Airbnb, hotel, ferry                           |
| `Investments`     | Syfe, Endowus, StashAway, Tiger Brokers, ETF, CPF invest |
| `Services`        | Singtel, StarHub, SP Group, insurance                    |
| `Family`          | school, tuition, childcare, kindergarten                 |
| `Miscellaneous`   | everything else (edit in Supabase / app afterwards)      |

### Pausing or Stopping

Run `removeTrigger()` in the Apps Script editor to stop the automatic sync.

---

## Credits

This project is a **Supabase-powered rewrite** of an original expense tracker built by [**Jxff-so**](https://github.com/jxff-so/expense-tracker).

**Original Version:** Uses Notion as the backend database  
**This Version (P-Accountant):** Replaces Notion with Supabase (PostgreSQL) for better performance, real-time sync, and simplified self-hosting.

All UI design, styling, and front-end structure are derived from the original Notion-based project. The backend integration, database schema, RLS policies, and Gmail Apps Script parser have been partially rewritten to work with Supabase.

> 🔗 **Want to use the original Notion version?**  
> Visit: [**Spendly with Notion**](https://jxff-so.github.io/expense-tracker/)

---

### Key Differences from Original

| Feature | Original (Notion) | This Version (Supabase) |
|---------|-------------------|-------------------------|
| Database | Notion API (requires proxy) | Supabase (direct connection) |
| Setup Complexity | Requires Cloudflare Worker | No proxy needed |
| Self-hosted | Yes (with Worker) | Yes (direct) |
| RLS Security | Notion's native permissions | PostgreSQL RLS policies |

---

**Built with:** HTML5, CSS3, JavaScript, Supabase, Google Apps Script  
**Design inspired by:** [Spendly](https://github.com/jxff-so/expense-tracker)
