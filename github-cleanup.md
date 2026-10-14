# DO THIS: declutter a GitHub account without deleting anything

> Paste this whole file to an agent and say: **do this.**

**Goal.** Move a user's stale repos into an archive organization so their personal
profile shows only current work — while every repo stays public and intact.
Delete nothing. Change no visibility.

## The two facts that make this work

- GitHub has **no "unlisted" state**: a public repo is on the profile, a private repo
  isn't. Archiving only changes *write* permissions, never *visibility*, so it cannot
  de-clutter a profile.
- An **org-owned** repo disappears from the personal profile but stays public, keeps its
  stars/issues/history, and old URLs redirect. So the lever is **ownership**, not settings.

## Ask the user first

1. GitHub username.
2. Which repos are **current** (stay on the personal account).
3. Archive org name (create it if it doesn't exist).

Do not guess the keep list. If the user has no preference, show candidates and ask.

## Steps

1. **Inventory** — `gh repo list <user> --limit 1000 --json name,description,visibility,isFork,isArchived,primaryLanguage,repositoryTopics,pushedAt`. Save to a file; report counts.
2. **Compute the transfer list** = all repos minus the keep list. **Print it and get a yes before transferring.**
3. **Verify the org** — `gh api user/memberships/orgs/<org>` (must be `admin`/`active`).
4. **Transfer** in a loop, public then private:
   `gh api repos/<user>/<repo>/transfer -X POST -f new_owner=<org>`
5. **Verify independently — do not trust the transfer response.** Re-list the org and
   confirm counts; confirm private repos are still private; hit one old URL and confirm
   it resolves to the new owner.
6. **Metadata (optional)** — add descriptions/topics so the archive is browsable. Read
   each repo's README first; never invent a description from the name.
7. **READMEs (optional)** — rewrite the personal profile README (repo named after the
   user) to point at the org; create `<org>/.github` with `profile/README.md`.
8. **Pins** — there is no API. Tell the user to pin manually in the profile UI.

## Gotchas that will bite you

- **Archived repos reject `PATCH description`** with `403 Repository was archived so is
  read-only`. Workaround: unarchive → patch → re-archive. (Topics *do* still work while archived.)
- The **transfer response's `.full_name` still shows the old owner** even on success.
  Verify by re-reading under the new owner.
- **Visibility casing differs:** `gh repo list --json visibility` → `PUBLIC`/`PRIVATE`;
  REST `.visibility` → `public`/`private`. Normalise before comparing.
- **`repositoryTopics` can be null** — guard `jq` with `(.repositoryTopics // [])`.
- **No public pinning API** — REST `/user/pinned_repositories` → 404; GraphQL
  `updateUserPinnedItems` → undefined. UI only. Pins survive transfers.

## Never

- Delete a repo.
- Change a repo's visibility.
- Transfer without showing the list and getting confirmation.

## Done when

The personal profile lists only the keep repos, the org holds the rest, private repos
are still private, and old URLs redirect.
