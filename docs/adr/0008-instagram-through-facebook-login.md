# Instagram through Facebook Login, on Page tokens

v1 reaches Instagram through the Facebook Login setup of the user's Meta app, calling `graph.facebook.com` with the token of the Page the Instagram account is linked to; it doesn't use Instagram Login or `graph.instagram.com`, though Meta now promotes them. Page tokens derived from a long-lived user token never expire, and in the live check they reached every Instagram call, publish, delete and DMs included, so daily use never logs in again. Facebook Login also adds media delete, ordered comments, collaborators and business discovery. The cost: the Instagram account must be linked to a Page, which setup checks, and an app holds only one Instagram setup, so adding Instagram Login later means a second app.

## Considered options

- **Instagram Login**: no Page needed, 60-day tokens refreshed after 24 hours, URL-only media, no media delete. Left in the fog; it may matter for DMs.
- **Unified Login through a system user**: its template defaults to ads permissions and an ad account and leaves out insights.
