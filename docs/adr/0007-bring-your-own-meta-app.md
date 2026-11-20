# Every user brings their own Meta app, kept in development mode

social-kit ships no app id. Each user creates one Business-type Meta app with the Pages, Instagram (Facebook Login), Threads and Messenger use cases, pastes our privacy policy into it, and keeps it in development mode. Development mode with Standard Access reaches every asset the user owns or manages, indefinitely, with no App Review, Business Verification or hosted service of ours; a shared app would make the project the party holding every user's tokens, review every permission, and run a public OAuth redirect. The cost is a one-time dashboard pass that `skit auth login` guides, and development mode's limits: DMs and private replies reach only people with a role on the app until the user takes their own app through App Review.

## Considered options

- **One shared social-kit app**: App Review per permission, Business Verification, a hosted redirect and a privacy policy covering every user's data.
- **Login for Devices on a shared app id**: still a shared app, with the same review and responsibility.
