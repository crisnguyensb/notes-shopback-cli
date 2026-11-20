# Let your coding agent find the cashback

You are about to buy something. You have a coding agent open in a terminal
anyway. It can read your code, run your tests, and open a pull request — but it
cannot tell you that the store you are about to buy from pays 6% cashback, or
make sure the purchase actually gets tracked.

`@shopback/cli` fixes that. It is a small command-line tool, `sb`, that gives
any agent on your machine a way to look up ShopBack cashback and open a store
through a tracked link.

It works today in **Australia, New Zealand, Singapore and the United States**.
Any other market stops with a clear error listing the ones that work, so if you
are not in one of those four, this is not useful to you yet.

## Getting it running

Two commands:

```sh
npm i -g @shopback/cli && sb agent setup
sb login
```

You need Node 22.23.4 or newer. Older versions are refused on purpose — that
release was the October 2026 security fix, and the same applies to 24.x and 26.x
below their patched versions.

`sb login` prints a URL and a short code. You open the URL in any browser, on
any device, and type the code. That is it. There is no API key to paste
anywhere, and no secret in a config file: your tokens go into your operating
system's keychain. `sb whoami` shows who you are signed in as, and `sb logout`
revokes access and clears the stored tokens.

The reason login works this way is that your agent often has no browser to open
— it might be on a server over SSH, in CI, or in a sandbox. Handing you a code
to approve somewhere else works everywhere.

## What `sb agent setup` does to your machine

This is the part worth understanding before you run it, because it writes into
your agent's configuration.

- **Claude Code** gets a skill file at `~/.claude/skills/shopback/SKILL.md`.
- **Codex** gets a block inside your global `~/.codex/AGENTS.md`. The block is
  marked, so the rest of that file is left alone.

If you set `CLAUDE_CONFIG_DIR` or `CODEX_HOME`, those are respected.

By default it sets up every agent it finds. `sb agent setup --agent codex` does
just one. Run it again any time to update what it installed.

To undo it:

```sh
sb agent remove              # all agents
sb agent remove --agent codex  # just one
```

`agent remove` takes out only what `agent setup` put in. Your own `AGENTS.md`
content stays.

Why this exists: an agent that installs a CLI and has no idea what it is for
will never use it. So the tool teaches your agents about itself. Until setup has
run, every other command keeps suggesting it, which means an agent that only
managed to install the CLI will finish the job on its own.

## Using it

In practice you do not type these yourself — you ask your agent, and it runs
them. But the shape is simple enough to follow:

```sh
sb search nike               # find the store and its merchant ID
sb offer 19160 --amount 150  # the current rate, tiers, caps, and what you'd earn
sb go 19160                  # start a tracked trip and open the store
```

`sb search` matches store and brand names. `sb offer` shows the active rate with
its tiers and caps, any upsize coming up, the store's terms, and — if you pass
`--amount` — the estimated cashback per tier.

`sb go` is the one that matters for getting paid. It starts a shopping trip and
opens the store in your default browser. Each call creates a new trip, so a
well-behaved agent will ask you first whether to open it here or just hand you
the link. If there is no browser to open, or you pass `--no-open`, you get the
link back instead.

Every one of these needs you to be signed in first.

## Which country you are in

Once you are signed in, your market comes from your ShopBack account. You do not
choose it per command.

The `--country` flag (and `SB_COUNTRY`) only applies while you are signed out,
where it defaults to `US`. If you pass a country that disagrees with the account
you are signed in as, shopping commands stop rather than quietly giving you the
wrong market's rates.

## Why your agent does not get confused

Two small things make a large difference in practice, and you will notice them
as the tool just working rather than your agent flailing.

**Every command prints one JSON document.** No tables, no colour, no spinners —
including when something fails. Your agent never has to guess whether it got a
result or an error message.

**Every response says what to do next.** Each one carries a `next` list: the
commands that make sense from here. After a search, that is the offer lookup for
the store it found. When you are not signed in, the error does not only say so —
it tells your agent to run `sb login`. Your agent learns the workflow from the
output instead of reading documentation.

Errors are also split by what you should do about them, not by what broke
internally. "You need to sign in" and "ShopBack refused this request, and
signing in again will not help" are different codes, because an agent that
cannot tell them apart will sit in a login loop forever.

## What is still rough

**Search is the weak spot.** It is good with store and brand names and bad with
product categories. "shirt" will find ZALORA, H&M and Uniqlo. "shoe" finds
nothing useful. Search for the brand or the store, not the thing you want to
buy.

**Rate limits do not tell you how long to wait.** When you hit one, the retry
time usually comes back empty, which means unknown rather than zero. An agent
that treats it as zero will keep hammering.

**Only two agents are supported by setup today**, Claude Code and Codex. Others
can still use the CLI; they just will not be configured automatically.

## Try it

```sh
npm i -g @shopback/cli && sb agent setup
sb login
sb search nike
```

Australia, New Zealand, Singapore and the United States, for now.

---

*These are my own notes on a tool I work on, not an official ShopBack
announcement. The official package is at
[npmjs.com/package/@shopback/cli](https://www.npmjs.com/package/@shopback/cli).*
