# Known limitations

What skit can't do, and why. Most of it is missing from Meta's APIs; the rest sits behind App Review, which a bring-your-own-app user in development mode doesn't have.

## All platforms

- **Bursts can get an account suspended.** A new Threads account was locked pending appeal after about 15 near-identical test posts and replies through the API in a few hours, then restored. Space publishes out and vary the text.
- **Local files on Instagram and Threads need `cloudflared`.** Both APIs publish only from a URL Meta fetches, so skit exposes the file through a temporary Cloudflare Quick Tunnel until Meta has it. Keep the machine online until the command returns. The file passes through Cloudflare on its way to Meta. Meta downloads it at your upload speed, so on a slow connection a large video can time out on Meta's side; tested up to 76 MB. Cloudflare calls Quick Tunnels "for testing and development", with no uptime guarantee, and if Meta's fetchers ever start honouring Cloudflare's injected robots.txt, this route stops working. Passing a URL you host always works. Facebook and DMs upload the file directly and don't need it.
- **HEIC photos** are refused before upload; Meta asks for JPEG. Convert first (macOS: `sips -s format jpeg IN.heic --out OUT.jpg`; elsewhere: `magick IN.heic OUT.jpg`), or host a JPEG.

## Instagram and Facebook messages

Unreachable without App Review:

- **Strangers' DMs.** In development mode, `dm list` shows only conversations with people who have a role on your app. Everyone else is left out without an error, older conversations included. Reaching customers needs App Review for Advanced Access, which also takes Business Verification.

Not in the API:

- **History.** Only the 20 most recent messages of a conversation can be read.
- **Unread on Instagram.** Instagram reports no unread state; `dm list --waiting` shows conversations whose last message is theirs. Reading through skit never marks anything read.
- **Read receipts, delivery, button taps and real-time messages.** Meta sends these only to webhooks, which skit doesn't run.
- **Group chats.**
- **Answering after 24 hours.** Meta refuses ordinary messages once the reply window closes. `--human-agent` extends it to 7 days, but Meta grants that only after review.

## Threads

Not in the API:

- **Scheduling.** No scheduled publish time. Run `skit th post create` from an outside scheduler (`at`, cron, launchd, or your agent's scheduled tasks).
- **Editing** a post or reply after publishing. Delete and republish instead.
- **Likes**, and **follower or following lists**.
- **Following.** `skit th profile follow @user` only opens Threads' follow page; you click Follow yourself.
- **DMs.**
- **Text styling in the main text.** Bold, italic, highlight, underline and strikethrough work only in a post's long text; the main text takes spoilers only.
- **Poll duration.** Polls can't be timed or closed.

Unreachable without App Review:

- **Location tagging.** In development mode, location search returns only places around Menlo Park, whatever you search for.
- **Quoting, reposting and replying** on other people's posts reach only the app's Threads Testers' posts. Your own posts and the replies under them are not affected.
- **Search** covers only your own posts.
- **Profile discovery** (`profile get @user`, `post list @user`) reaches only @threads, @meta, @instagram and @facebook.
- **Mentions** (`mention list`) shows only mentions made by the app's Threads Testers, per Meta's docs.
- **Other people's posts** can't be read by id. Replies to your own posts are not affected: everyone's replies show.

Other:

- **Sharing to Instagram Stories.** The API has it, but the Threads use case doesn't offer its permission, so skit can't request it.
- **Hiding replies** works only on top-level replies to your posts, not nested ones or your own.
- **Geo-gating** (`--countries`) works only for accounts Meta marks eligible.
