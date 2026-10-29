# Connecting to Yandex Direct

The commands read `config/.env` themselves. Do not print this file, the token or the
Authorization header into the chat, the debug log or the command line.

If `.env` is already set up, use it. For a new connection, copy
`.env.example` to `.env` and fill it in with an editor:

- `YANDEX_DIRECT_TOKEN` - the OAuth token of a user with access to the Direct API.
- `YANDEX_DIRECT_ACCOUNT` - the login of the default ad account;
  below you can see where to get it and how to pick another account.
- `YANDEX_DIRECT_LOCALE` - the language of API messages: `ru`, `en` or `tr`.
- `YANDEX_DIRECT_USE_OPERATOR_UNITS` - whose points to spend: `auto`, `never`
  (the client's) or `always` (the agency's). `auto` is usually enough.

`ACCOUNT` needs the login of the client who owns the ads. You can find it
in the output of `accounts.py` or take it from the `ulogin` parameter in the URL of
the open account. For example, for
`https://direct.yandex.ru/dna/grid/campaigns?ulogin=client-shop` the setting is:

```dotenv
YANDEX_DIRECT_ACCOUNT=client-shop
```

Here `client-shop` is an example: replace it with your own login. A company name,
a campaign ID or an agency login instead of the client login will not work. This setting
picks an account among those the token can already reach; it does not grant access to someone else's account.

Advertising commands choose the account in this order:

1. An explicit `--account` in the current command.
2. The saved choice from `accounts.py --use`.
3. The only non-archived account, if there is nothing to choose from.
4. `YANDEX_DIRECT_ACCOUNT` from the settings.

Environment variables take priority over values in `.env`. The `ACCOUNT` field
can be left empty: your only account will be detected automatically,
and with several accounts use `--account` or `--use`.

For example, with several accounts available and no saved choice, the first
command below reads the campaigns of `client-shop` from the settings, the second reads those of another
client for this run only, and the third remembers that client for later commands:

```bash
uv run scripts/campaigns.py
uv run scripts/campaigns.py --account client-service
uv run scripts/accounts.py --use client-service
```

After the third command, `campaigns.py` without `--account` will pick `client-service`,
even if `.env` still says `client-shop`. For parallel work, pass
`--account` explicitly so the commands do not depend on the shared saved choice.

Checking the connection from the skill directory:

```bash
uv run scripts/whoami.py
uv run scripts/accounts.py
```

`whoami.py` checks the connection using `--account` or `ACCOUNT` from the settings and
does not use the saved `--use` choice. Without these values it checks your own
advertiser account or agency access without picking a separate client.

If you have no access yet, open [API_ACCESS.md](API_ACCESS.md): it has the pages
for registering the application and submitting the request, filling-in examples and how to get a token.

The extra profile `--env test_cabinet` uses
`YANDEX_DIRECT_ACCOUNT_TEST_CABINET` and, if set,
`YANDEX_DIRECT_TOKEN_TEST_CABINET`. Without a separate token the main one is used.
This is a regular account with real ads, not a sandbox.
This profile does not inherit the main `YANDEX_DIRECT_ACCOUNT`. The choice made with
`accounts.py --use` is saved separately for each profile.

Of `config/`, only the instructions `README.md`, `API_ACCESS.md` and
the empty sample `.env.example` go into Git. Tokens, selected accounts, cache and logs
stay local.
