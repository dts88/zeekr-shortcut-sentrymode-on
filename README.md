# Zeekr Sentry Mode Auto-On

When you park your Zeekr anywhere other than the places you choose (home, office, ...), this program turns sentry mode
on for you within a few minutes.

- **Free:** it runs in your own Cloudflare account, on the free plan.
- **Nothing to keep running:** it runs on Cloudflare's servers, so your computer and phone can be off.
- **No code:** you set it up in your browser by clicking and filling in a few boxes. It takes about 15 minutes.
- **Easy to upgrade:** a new version is one click on GitHub, and your settings stay.

**How it works**

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/images/overview-dark.svg">
  <img alt="A Worker in your Cloudflare environment keeps the second (shared) Zeekr account encrypted and talks to Zeekr's servers over an encrypted connection; the servers pass the sentry command to your car." src="docs/images/overview-light.svg">
</picture>

The program runs in **your own** Cloudflare account. Your second Zeekr account's email and password, and your places,
are stored there (the password always encrypted). Every 5 minutes it signs in to Zeekr's servers over an encrypted
(HTTPS) connection, the same way the Zeekr app does, reads where the car is, its gear, speed and battery level, and
asks for sentry mode when needed; Zeekr's servers pass that on to the car. There is no server of the author's in
between, and nobody else can see your settings.

> This is an unofficial, personal project, not affiliated with Zeekr. It works through the same service the Zeekr app
> uses, so it may stop working for a while if Zeekr changes that service.
> Tested with a Singapore account. Accounts from Zeekr's Southeast Asia, Middle East and Latin America regions should
> work as they are. A European account works once you add the European app's keys yourself (see
> [European accounts](#european-accounts)); that is not yet tested on a real car.

**Contents**

1. [What it does](#what-it-does)
2. [Before you start: four things to prepare](#before-you-start-four-things-to-prepare)
3. [Set up, step by step](#set-up-step-by-step)
4. [Check that it works](#check-that-it-works)
5. [Everyday use](#everyday-use): logs, changing settings, pausing, upgrading, uninstalling
6. [FAQ](#faq)
7. [Privacy and security](#privacy-and-security)

---

## What it does

Every 5 minutes the program checks your car:

| Your car is ... | The program ... |
|---|---|
| being driven | does nothing |
| parked within one of your places (for example at home) | does nothing |
| parked elsewhere, battery at least 20%, sentry mode off | **turns sentry mode on**, trying up to 6 times per parking (once every 5 minutes) |
| parked, and sentry mode was on during this parking but has been turned off (by you or by the car) | leaves it off |

Parked means in P. While the car is still switched on (for example with you in it) it does not take
the command, so the program waits until the car is switched off, without using up its tries. It never turns sentry
mode off.

---

## Before you start: four things to prepare

### 1. A second Zeekr account that your car is shared with

The program has to sign in to a Zeekr account to check the car and turn sentry mode on. **Use a second account, not the
one in your phone's Zeekr app:** when an account signs in somewhere else, the app is signed out.

1. Register a new Zeekr account with a different email address.
2. In the Zeekr app, signed in with your **main** account, open **Vehicle & Security** → **Share Vehicle** (1) →
   **Create Sharing** (2), and share the car with the second account's email address, for an unlimited time.
3. Sign in to the Zeekr app with the **second** account and accept the shared car. Back in the main account, the
   sharing list now shows **Accepted**, **Started** and **Unlimited time** (3).
4. While signed in with the second account, check that you can see the car and its sentry mode switch, then sign out.
   Don't use the second account in the app after that.

<p>
  <img alt="Vehicle &amp; Security, then Share Vehicle" src="docs/images/zeekr-app-vehicle-security.jpg" width="300">
  &nbsp;
  <img alt="Share Vehicle: Create Sharing; the sharing list shows Accepted, Started, Unlimited time" src="docs/images/zeekr-app-share-vehicle.jpg" width="300">
</p>

### 2. The places where sentry mode should not turn on (up to 10)

Near these places the program never turns sentry mode on: home, office, and so on.

1. On a computer, open [Google Maps](https://www.google.com/maps), find the place and **right-click** it.
2. The first line of the menu is two numbers (latitude, longitude). Click it to copy them.

   ![Google Maps: right-click a place, the first line is its latitude and longitude](docs/images/google-maps-coordinates.jpg)

3. Write each place as `name, latitude, longitude, radius`, with the radius in metres. Separate places with a semicolon:

```
Home, 1.3521, 103.8198, 600; Office, 1.2840, 103.8510, 600
```

- **The name** only appears in the program's log, so you can tell the places apart.
- **The radius** can be left out; it is then 600 metres. In underground car parks the car's reported position often
  drifts by a few hundred metres, and a radius that is too small makes "at home" look like "away". If there is a place
  nearby where you do want sentry mode (a shopping mall car park, say), make the radius smaller.
- **The exact spot:** where the car reports itself can differ from the map, in underground car parks especially. Once
  the program runs, you can use the position the car itself reports: see
  [Find the car's exact position](#find-the-cars-exact-position).
- Commas and semicolons typed with a Chinese or Japanese keyboard (`，` `；`) are fine too.
- At least one place is required. If you really want none, write `none`.

### 3. A Cloudflare account (free)

Go to https://dash.cloudflare.com/sign-up , sign up with an email address and a password, then click the link in the
verification email. New accounts are on the free plan; no credit card is needed.

### 4. A GitHub account (free)

Go to https://github.com/signup and sign up. You keep your own copy of this program there, called a *fork*. Cloudflare
runs the program from your fork, and upgrading later is one click there.

---

## Set up, step by step

Use a computer. It takes about 10 minutes. GitHub and Cloudflare sometimes reword their pages, so what you see may
differ slightly from the pictures below.

### Step 1: Make your copy on GitHub (fork)

1. Sign in to GitHub, then open https://github.com/dts88/zeekr-shortcut-sentrymode-on/fork (or click **Fork** at the
   top right of this page).
2. On **Create a new fork**, keep everything as it is. Above all, keep the **Repository name**
   `zeekr-shortcut-sentrymode-on`: Cloudflare names the program after it, and the program only deploys under that name.
3. Click **Create fork**.

Your GitHub account now has its own copy, `<your account>/zeekr-shortcut-sentrymode-on`. Like this page, it is public
and contains only the program: your settings go into Cloudflare, never into GitHub.

### Step 2: Connect Cloudflare to your fork

1. In the [Cloudflare dashboard](https://dash.cloudflare.com), open **Compute** → **Workers & Pages** in the left menu
   (1) and click **Create application** (2).

   ![Workers & Pages, then Create application](docs/images/create-application.jpg)

2. Click **Continue with GitHub**.

   ![Create an app: Continue with GitHub](docs/images/import-github.jpg)

3. The first time, GitHub opens and asks you to install **Cloudflare Workers and Pages**. Choose
   **Only select repositories**, select `zeekr-shortcut-sentrymode-on`, and click **Install & Authorize**. You come
   back to Cloudflare.
4. Under **Select a repository**, click `zeekr-shortcut-sentrymode-on` (1), then **Next** (2). If you have many
   repositories, type `sentry` in the search box first.

   ![Select a repository: your fork, then Next](docs/images/import-select-repo.jpg)

   Not in the list? Cloudflare may use only some of your repositories. On GitHub, click your picture (top right) →
   **Settings** → **Applications** → **Cloudflare Workers and Pages** → **Configure**, add
   `zeekr-shortcut-sentrymode-on` under **Repository access** and click **Save**.

### Step 3: Deploy

On **Set up your application**:

1. **Project name:** keep `zeekr-shortcut-sentrymode-on` (1). With any other name, the deployment fails.
2. Leave everything else as it is and click **Deploy** (2).

![Set up your application: keep the project name, then Deploy](docs/images/import-set-up.jpg)

Cloudflare builds and deploys the program, which takes 1–3 minutes. It has then:

- created the program, a Worker named `zeekr-shortcut-sentrymode-on`,
- created its storage space, `zeekr-shortcut-sentrymode-on-state`, for the sign-in session and the current parking,
- scheduled it to run every 5 minutes,
- linked it to your fork: a change there is deployed by itself (that is how upgrades work).

The program now runs, but until you add your settings in the next step, its log only says
`missing: ZEEKR_EMAIL, ZEEKR_PASSWORD, GEOFENCES`.

### Step 4: Add your three settings

1. Open the Worker: **Compute** → **Workers & Pages** → `zeekr-shortcut-sentrymode-on`. Open **Settings** (1), and
   under **Runtime variables and secrets** click **+ Add variable** (2).

   ![Settings, then Add variable](docs/images/settings-add-variable.jpg)

2. Fill in the first row (1): the name in **Key**, the value in **Value**, and tick **Secret** where the table says
   so. Click **+ Add** (2) for the next row, until you have all three:

   | Key | Value | Secret |
   |---|---|---|
   | `ZEEKR_EMAIL` | the second Zeekr account's email address | don't tick |
   | `ZEEKR_PASSWORD` | that account's password | **always tick it** |
   | `GEOFENCES` | your places, for example `Home, 1.3521, 103.8198, 600` | don't tick |

3. Click **Add variable and deploy** (3).

   ![Key, Value and Secret; + Add for the next row; then Add variable and deploy](docs/images/add-variables.jpg)

Type the names exactly as above, in capital letters. With **Secret** ticked, a value is stored encrypted and can't be
viewed again, not even by you; you can still replace it (see
[Change your places or password](#change-your-places-or-password)).

---

## Check that it works

### Read the log

1. In the Cloudflare dashboard, open **Compute** → **Workers & Pages** in the left menu and click
   `zeekr-shortcut-sentrymode-on`.

   ![Workers & Pages, then zeekr-shortcut-sentrymode-on](docs/images/workers-list.jpg)

2. Open the **Observability** tab. Each run adds a few lines. Lines starting with `[v…]` come from the program;
   lines showing `*/5 * * * *` only mark that a scheduled run happened.

   ![The Observability tab: the program's lines start with the version](docs/images/logs.jpg)

A new schedule can take up to 15 minutes to start. The first run looks like this (`[v0.4.4]` is the program's version):

```
[v0.4.4] logging in · zeekr-sentry 0.4.4 · https://github.com/dts88/zeekr-shortcut-sentrymode-on · based on Fryyyyy/zeekr_ev_api (MIT)
[v0.4.4] logged in (SG account, region SEA), 1 car(s)
[v0.4.4] car …123456 gear=P speed=0km/h usage=1 battery=80% sentry=off 120 m from "Home" -> none: inside geofence "Home"
```

The last line is the result of each check:

| Part | Meaning |
|---|---|
| `car …123456` | the last 6 characters of the car's VIN |
| `gear=P` | the car is in P (another value is the number the car reports for another gear) |
| `speed=0km/h` | the car's speed (shown only: a parked car sometimes reports one) |
| `usage=1` | the car's power state as Zeekr reports it: `13` or `4` means switched on and in use |
| `battery=80%` | battery level |
| `sentry=off` | sentry mode is currently off |
| `120 m from "Home"` | distance to the nearest of your places |
| `-> none: inside geofence "Home"` | what the program decided, and why: within Home, so nothing to do |

### Try it in the car

Park somewhere outside your places and put the car in P. Within 10 minutes, open the Zeekr app: sentry mode should be
on. The log shows:

```
[v0.4.4] car …123456 gear=P speed=0km/h usage=1 battery=80% sentry=off 2400 m from "Home" -> enable: attempt 1/6
[v0.4.4] car …123456 sentry start sent
```

---

## Everyday use

### Read the log

Cloudflare dashboard → **Compute** → **Workers & Pages** → `zeekr-shortcut-sentrymode-on` → **Observability**.

### Change your places or password

1. Cloudflare dashboard → **Compute** → **Workers & Pages** → `zeekr-shortcut-sentrymode-on` → **Settings**.
2. Under **Runtime variables and secrets**, click the edit (pencil) button next to the setting, for example `GEOFENCES`.

   ![Settings, then the pencil next to the setting](docs/images/settings-secrets.jpg)

3. A secret (the password, and any other setting you ticked **Secret** for) cannot be viewed again: click
   **Rotate**, type the whole new value (not just the part you are changing), then click **Deploy**. A setting
   without **Secret** shows its value: change it in place, then click **Deploy**. (**Save version** alone does not
   put the change into use.)

   ![Rotate, type the new value, Deploy](docs/images/secret-rotate.jpg)

### Find the car's exact position

To set a place exactly where your car reports it is parked:

1. Park there, in P, and wait for one check (up to 5 minutes).
2. Cloudflare dashboard → **Storage & databases** → **Workers KV** → click `zeekr-shortcut-sentrymode-on-state`
   (`zeekr-sentry-state` if you set it up by pasting the file), then open **KV Pairs**.
3. Find the key that starts with `parking:` (one per car). Its **Value** looks like
   `{"position":{"lat":1.35318,"lon":103.8198},"attempts":0,"seenOn":false}`: `lat` and `lon` are where the car
   reported itself when this parking began.
4. Put them in `GEOFENCES`, for example `Home, 1.35318, 103.8198, 600` (see
   [Change your places or password](#change-your-places-or-password)).

The `parking:` key is there only while the car is parked; it goes when the car drives off. Leave the entries as they
are: `session` is the program's sign-in.

Thanks to R. Manochvarma, who found this method. A place set from the position the car itself reports is the most
exact one you can get.

### Pause and resume

On the Worker's **Settings** page, under **Trigger events**:

![Trigger events: the bin pauses, Add resumes; Delete removes the whole Worker](docs/images/trigger-events.jpg)

- **Pause:** delete the cron trigger that runs every 5 minutes (the bin icon, 1). An upgrade turns it back on.
- **Resume:** click **+ Add** (2). On the **Schedule** tab, set *Execute Worker every* to **Minute(s)** and change the
  number to **5** (it starts at 30), then click **Add**.

  ![Execute Worker every 5 Minute(s), then Add](docs/images/cron-add.jpg)

### Upgrade to a new version

New versions are published on this page. The `[v…]` at the start of each log line is the version you are running, and
once a day (just after midnight UTC) the log says `version … is available …` when there is a newer one.

1. On GitHub, open your fork, `https://github.com/<your account>/zeekr-shortcut-sentrymode-on`. When there is a new
   version, it says *This branch is … commits behind dts88/zeekr-shortcut-sentrymode-on:main*.
2. Click **Sync fork**, then **Update branch**.

Within a few minutes, Cloudflare deploys the new version by itself. Your settings, the storage space and the schedule
stay as they are, and the next log lines start with the new version.

If GitHub offers **Discard … commits** instead of **Update branch**, something in your fork was changed; discarding it
puts your fork back to this page's version.

### Uninstall

1. On the Worker's **Settings** page, under **Danger zone**, click **Delete** next to *Delete Worker* (3 in the picture
   above).
2. Cloudflare dashboard → **Storage & databases** → **Workers KV**: delete `zeekr-shortcut-sentrymode-on-state` from the
   **⋯** menu on its row.

   ![Workers KV: the ⋯ menu on the storage space's row](docs/images/kv-list.jpg)

3. On GitHub, delete your fork: open it → **Settings** → **Delete this repository** (at the bottom).
4. If nothing else of yours uses it, take back Cloudflare's access to GitHub: on GitHub, click your picture (top right)
   → **Settings** → **Applications** → **Cloudflare Workers and Pages** → **Configure** → **Uninstall**.
5. In the Zeekr app, stop sharing the car with the second account.

---

## FAQ

### Does it cost anything?

No. Cloudflare's free plan allows 100,000 requests a day; this program runs about 300 times a day. GitHub is free too.

### The deployment failed

The build log on that page says why. The usual causes:

- **The name:** the Cloudflare project must be called `zeekr-shortcut-sentrymode-on`.
- **A storage space left over from an earlier install:** the log says a namespace with the title
  `zeekr-shortcut-sentrymode-on-state` already exists. Delete it under **Storage & databases** → **Workers KV**.

Then delete the Worker (see [Uninstall](#uninstall), step 1) and do [Step 2](#step-2-connect-cloudflare-to-your-fork)
and [Step 3](#step-3-deploy) again.

### My phone's Zeekr app keeps getting signed out

`ZEEKR_EMAIL` is your main account. Change it to the second account's email and password (see
[Change your places or password](#change-your-places-or-password)).

### Sentry mode was turned on while the car was at home

Look at `… m from "Home"` in that log line. If the distance is larger than your radius, either the coordinates are off,
the radius is too small (underground car parks drift), or `GEOFENCES` is not written correctly.

### I parked away from home and sentry mode did not turn on

Look at the reason after `-> none:` in the log:

| Reason in the log | Meaning |
|---|---|
| `inside geofence "…"` | the car is within one of your places |
| `battery 15% below 20%` | battery is below 20%, so sentry mode stays off to protect it |
| `sentry turned off during this parking, respecting it` | sentry mode was on during this parking and was turned off, so it stays off |
| `gave up after 6 attempts` | 6 tries during this parking (one every 5 minutes) did not turn it on; the count starts again at the next parking |
| `not parked (gear 3)` | the car is not in P (3 is the number the car reports for its gear) |
| `gear unknown` | Zeekr did not report the car's gear; the parking is kept as it was |
| `car in use (usage 13)` | the car is parked but still switched on (someone is in it); it tries once the car is switched off |

### Other log messages

| Message | Meaning |
|---|---|
| `signed out: the account logged in elsewhere …, logging in again` | the second account was used somewhere else (the Zeekr app, or another copy of this program), so the program signs in again. If this happens often, check that nothing else uses the second account |
| `version … is available …` | a newer version is out: see [Upgrade to a new version](#upgrade-to-a-new-version) |
| `token expired, logging in again` | the sign-in expired and the program signs in again. Occasional ones are normal |
| `missing: …` | a setting is missing; add it as in [Step 4](#step-4-add-your-three-settings) |
| `geofence 2 should look like …` | the 2nd place is not written correctly |
| `sentry start failed: …` | Zeekr refused the command or did not answer in time; it counts as one of the 6 tries |
| `0 car(s)` | the second account has no car: share the car with it and accept the share; the program asks again every 5 minutes |
| `no Zeekr account with this email …` | the email is wrong. `… a European account needs the keys of the European Zeekr app`: see [European accounts](#european-accounts) |
| `Zeekr does not accept the app keys …` | the keys you added (see [European accounts](#european-accounts)) are wrong or out of date |
| `login needed but held back …` | the program signed in less than 30 minutes ago and waits before signing in again, so the account is not locked. If it keeps appearing, check the password |

### European accounts

Zeekr's European servers only accept the keys of the European Zeekr app (`com.zeekr.overseas`), and nobody publishes
those, so this program cannot include them. The program finds your account's region by itself; for a European
account, add the six keys of **your own** European app as secrets, the same way as the three settings in
[Step 4](#step-4-add-your-three-settings):

| Key | Where it comes from |
|---|---|
| `HMAC_ACCESS_KEY`, `HMAC_SECRET_KEY`, `PASSWORD_PUBLIC_KEY` | the app's installation file, with [zeekr_key_extractor](https://github.com/Wysie/zeekr_key_extractor) (`--region EU`) |
| `PROD_SECRET`, `VIN_KEY`, `VIN_IV` | read from the running app on your phone; see [openzeekr](https://github.com/borconi/openzeekr) and zeekr_key_extractor |

Once they are added, the log shows `logged in (NL account, region EU)` (with your country). This is new and not yet
tested on a European car: please report whether it works.

### I have two cars

Share both with the second account; the program checks each one.

---

## Privacy and security

- **Your Zeekr password** is stored as an encrypted secret in **your own** Cloudflare account; the email and your
  places are visible only to you, on the Worker's Settings page. Nobody else, including the author, can see them.
- **What the storage space holds:** a live Zeekr sign-in, which can find and control the car. Anyone who can change
  Workers in your Cloudflare account, or the code in your fork, could also get the password. Keep both accounts to
  yourself, with two-factor sign-in turned on. **Sync fork** brings in whatever this repository contains; **Compare**
  there shows the changes first.
- **Your fork on GitHub** is public, like this page, and contains only the program, none of your settings.
- **Cloudflare's access to GitHub** covers only the repositories you chose in Step 2: your fork.
- **The log** shows how far the car is from your places (by name), its battery level and so on, but never the email,
  the password or the coordinates. Keep that in mind before sharing screenshots of it.
- Give the second Zeekr account a password you don't use anywhere else.

---

## No GitHub account?

You can also set it up with only a Cloudflare account: create a Worker in the dashboard, paste in one file and set a
few things by hand. Upgrading then means pasting in the new file. See [docs/setup-manual.md](docs/setup-manual.md).

---

## Credits and license

MIT license, by [dts88](https://github.com/dts88). Based on:

- [Fryyyyy/zeekr_ev_api](https://github.com/Fryyyyy/zeekr_ev_api) (MIT): the Zeekr service, sign-in and request signing
- [Fryyyyy/zeekr_homeassistant](https://github.com/Fryyyyy/zeekr_homeassistant) (MIT): the sentry command and status fields
- [wysie/zeekr_key_extractor](https://github.com/wysie/zeekr_key_extractor) (MIT): constants from the public Zeekr app
- [borconi/openzeekr](https://github.com/borconi/openzeekr) (MIT): the European servers

For developers: [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md)
