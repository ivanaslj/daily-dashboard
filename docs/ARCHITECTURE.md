# Architecture

How the pieces fit. No source code here, just the shape of it.

## The rule everything follows

The page never needs a reload and never scrolls. Every panel is sized to the window. When a list is too long, the panel trims it and adds a "View all" link instead of growing.

## Server

A Python Flask app.

Each kind of data is one module with one function, `get_data()`. The page asks `/api/<module>` for each panel on its own, so the panels load in parallel. A broken Canvas feed empties the Canvas panels. Sleep, gym and the plan don't care.

| Module | Where the data comes from |
|---|---|
| Canvas | the school's private calendar feed (.ics) |
| Calendar | a private Google Calendar feed (.ics), recurring events expanded |
| Plan | Claude, given today's events, free time and deadlines |
| Tonight | a plan typed by the user, pinned for one day |
| Personal tasks | Google Tasks, through OAuth |
| Sleep | a form, an iPhone Shortcut, or WHOOP |
| Gym | check-ins and a weekly routine |
| Countdowns | dates added in the dashboard |
| Weather, news | public APIs and RSS feeds |

Feeds are cached for a few minutes in memory. Today's Plan is cached for the whole day, so the API is called once a day, not once per refresh.

## Page

Plain JavaScript, split by panel. Each panel is a small object that knows its data source, its title, and how to draw itself small (the tile) and large (the detail view).

A shared fitting step runs after every draw. It removes list items from the end until nothing overflows, then shows how many were hidden.

Charts are SVG built by hand: bars, rings, a radar for class load, a heatmap for the gym.

## Look

- Frosted glass panels with a 1px highlight and a deep shadow.
- A liquid-metal background in WebGL, capped at 30 frames per second and paused when the tab is hidden.
- Three fonts. Serif for numbers and names, sans for reading, narrow caps for labels.
- Every color is a CSS variable. A theme is just a different set of values, so switching themes changes no markup.

## Profiles and privacy

Sign-in decides who you are. Everything after that is scoped to you:

- Every database row carries a user id, and every query filters on it.
- Each profile's feed links and keys are server-side settings, never sent to the page.
- Files like study guides are served only from the signed-in profile's own folder.
- Browser settings (theme, brightness, timer) are stored under the profile's own name.

## Where it's going

| Today | Next |
|---|---|
| Runs on a laptop | Runs on Render |
| SQLite file | Supabase Postgres with row level security |
| Profile picker | Email login |
| Separate small servers for Spotify, WHOOP and Google Tasks | One server, connections per user |
| Study files on disk | Private cloud storage |

With row level security, even a bug in the server can't leak data. The database checks every row against the signed-in user before it answers.
