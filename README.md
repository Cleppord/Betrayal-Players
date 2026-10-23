# Betrayal

A black-and-red, phone-friendly stat tracker for playing *Betrayal at House on the Hill* with friends. Everyone opens the same link on their own phone, picks their character, and drags their Speed, Might, Sanity and Knowledge tracks. Everyone else sees the changes the moment they happen.

It's a single static `index.html` hosted on GitHub Pages, with [Supabase](https://supabase.com) handling the live data, photos and admin login.

## Features

- **Who are you?** start screen: pick your character to join the game.
- **Live lobby** showing only the players currently in the game, with each one's current SPD / MGT / SAN / KNW updating in real time.
- **Drag-to-set stat tracks** (8 positions per trait plus a skull). Tap a number or drag up and down. Dashed outline marks the starting spot.
- **Watch-only mode** for other players' pages. You can only move your own stats.
- **Profile photos** uploaded straight from a phone's camera or gallery, visible to everyone.
- **About pages** for each character, with "Unknown Character spooky!" for anyone without a bio.
- **Admin login** to edit bios and track values live. Players see the changes instantly.
- **Balance checker** in the admin editor for the custom-character rules (total 150, starting values sum 15, and so on).

## How it works

| Piece | What it does |
| --- | --- |
| `index.html` | The whole app: UI, logic and styling in one file. |
| Supabase Postgres | Two tables: `characters` (bio and track values, admin-written) and `live` (current positions, photo, in-game flag, writable by anyone). |
| Supabase Realtime | Pushes changes in either table to every open phone. |
| Supabase Storage | A public `photos` bucket for profile pictures. |
| Supabase Auth | Email and password login for the admin only. |

A player joining sets `active = true` on their row in `live`. **Leave game** sets it back to false, and the admin's **Clear all players** resets everyone.

## Setup

### 1. Create the Supabase project

1. Create a free project at [supabase.com](https://supabase.com).
2. Open **SQL Editor**, paste in the contents of [`supabase-setup.sql`](supabase-setup.sql) and run it. The script is safe to run more than once.
3. Open **Authentication → Users → Add user → Create new user**. Enter your admin email and password and tick **Auto Confirm User**.
4. In the Auth settings, turn off **Allow new users to sign up** so you're the only account that can ever log in.

### 2. Add your keys

Open `index.html` and fill in the two lines near the top of the script:

```js
const SB_URL='https://YOUR-PROJECT.supabase.co';
const SB_KEY='YOUR-ANON-PUBLIC-KEY';
```

- **Project URL:** Project Settings → Data API.
- **Key:** Project Settings → API Keys → **Legacy API keys** tab → the `anon` key (a long string starting with `eyJ`). The page pins an older `supabase-js` release, which doesn't connect live with the newer `sb_publishable_` keys.

The `anon` key is designed to be public. Row-level security in the SQL file is what protects your data.

### 3. Deploy to GitHub Pages

1. Push `index.html` (and this README) to a GitHub repo.
2. In the repo, go to **Settings → Pages**, choose the branch and root folder, and save.
3. Open the published link on your phone.

### 4. Load the characters

Log in as admin (the **Admin** button on the start screen), open the lobby, and tap **Load defaults** once. This writes the built-in character data to the database.

## Using it

**Players**
1. Open the link and tap your character on **Who are you?**
2. You land in the lobby. Tap your card to open your stats and drag the tracks.
3. Tap a friend's card to watch their stats live.
4. On the **About** tab, tap **Upload photo** to set your picture.
5. Tap **Leave game** when you're done. Closing the tab keeps you in the game.

**Admin**
- Log in, open any character (including ones not in the game) and tap **Edit** to change the bio, track values and starting positions.
- **Clear all players** resets the lobby between games. **Load defaults** restores the built-in characters.

## Customizing

- **Characters and defaults:** edit the `DEF` object and the `ORDER` list in `index.html`.
- **Colors:** change the CSS variables at the top of the stylesheet (`--red`, `--deep`, `--panel`, and so on).
- **Fonts:** Cinzel Decorative and Spectral from Google Fonts.

## Security notes

- Only the logged-in admin can change bios and track values. This is enforced by the database, not just the page.
- Anyone with the link can join, move stats and upload photos. The "watch-only for other players" rule is a courtesy in the page, not a lock, so this is built for a group of friends and not for strangers.
- Don't commit your `service_role` key anywhere. The site only ever needs the public `anon` key.

## Troubleshooting

The small status label in the top-right corner shows the connection state.

| You see | What to check |
| --- | --- |
| `● Live` | Everything is working. |
| `Failed to fetch` | `SB_URL` is wrong, the project is paused, or something is blocking `supabase.co`. |
| `CHANNEL_ERROR` or stuck reconnecting | Use the legacy `anon` key. Re-run `supabase-setup.sql` to make sure realtime is on for both tables. |
| `Invalid login credentials` | The admin user doesn't exist or the email or password doesn't match. Re-create it in Authentication → Users with **Auto Confirm User** ticked. |
| Can't join, or "column active does not exist" | Re-run `supabase-setup.sql` in full. |

## Built with

Plain HTML, CSS and JavaScript, plus [`@supabase/supabase-js`](https://github.com/supabase/supabase-js) loaded from a CDN. No build step.
