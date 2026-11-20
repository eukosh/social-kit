# Official APIs only

social-kit talks only to Meta's documented Graph and Threads APIs, with tokens from the user's own app: never the private mobile or web APIs, scraping or session cookies. Every gap that leaves (no Threads DMs or scheduling, no strangers' DMs before App Review, no mentions feed) is listed in `docs/known-limitations.md`, not filled another way. Unofficial routes break Meta's terms, risk checkpoints and suspensions on the user's real accounts, and break without notice. Read-only tools such as agent-reach take them; social-kit borrows their ergonomics, never their routes.
