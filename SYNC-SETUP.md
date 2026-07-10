# Automatic sync — one-time setup

The app can keep the schedule identical across devices (Mac + iPhone) with no
hosting and no logins. It works by storing one copy of the schedule in a free
Firebase Realtime Database, at a secret unguessable path. Whoever knows the
secret code can read and write the schedule — the code is the key, exactly like
the existing transfer links. Nobody ever signs in from the app.

These same instructions appear inside the app (Set-Up → Automatic sync →
"Set up sync"), but here they are with more detail.

## 1. Create the free database (~5 minutes, done once)

1. Go to <https://console.firebase.google.com> and sign in with any Google
   account (this account is only used to own the database — the family never
   sees it).
2. Click **Create a project**. Name it anything, e.g. `carolyn-care`.
   Turn **off** Google Analytics if asked. Free "Spark" plan — no credit card.
3. In the left menu: **Build → Realtime Database → Create database**.
   Pick the United States location and **Start in locked mode**.
4. Open the **Rules** tab, replace the contents with the rules below, and
   press **Publish**:

   ```json
   {
     "rules": {
       "care": {
         "$code": {
           ".read": true,
           ".write": true,
           ".validate": "newData.hasChildren(['rev','updatedAt','state']) && newData.child('state').isString()"
         }
       }
     }
   }
   ```

   These rules mean: nothing in the database can be listed or browsed, but
   anyone who knows a full secret code can read/write that one schedule.
   The 128-bit random code is the password.
5. On the **Data** tab, copy the database address — it looks like
   `https://carolyn-care-default-rtdb.firebaseio.com`.

## 2. Turn sync on (on the device that has the current schedule)

Open the app → **Set-Up** tab → **Automatic sync** → **Set up sync (first
device)** → paste the database address → **Turn on sync**. The schedule is
pushed up and the card shows "✓ Synced".

## 3. Connect the other devices (either way works)

- **Easiest:** on the synced device, copy the **transfer link**
  (Set-Up → "Move to another device") and text/AirDrop it to the other device.
  Opening it and tapping "Load this schedule" both installs the schedule and
  joins the sync automatically.
- **Or:** copy the **sync code** (Set-Up → Automatic sync → "Copy sync code")
  and on the other device use Set-Up → Automatic sync → **Connect with a sync
  code**.

That's it. From then on, edits made anywhere show up everywhere by themselves —
live while the page is open, or on the next open/focus otherwise.

## How conflicts are handled

If two devices are edited while one is offline, the copy with the **newest
edit** wins when they reconnect. The losing copy is kept on that device under
the localStorage key `carolyn-care-v4-pre-sync` as an emergency backup, and
transfer-link imports are kept under `carolyn-care-v4-pre-import`.

## Costs and limits

The Firebase free tier includes 1 GB storage and 10 GB/month download. The
whole schedule is a few KB compressed; a family using this heavily stays
thousands of times below the limits. No card on file means it can never bill.
