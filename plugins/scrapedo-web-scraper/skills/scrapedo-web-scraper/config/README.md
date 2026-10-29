# Setting up the Scrape.do token

## Where to get a token

1. Sign up at https://scrape.do
2. In your account dashboard, copy the API token

## Where to put it

Create a `token.txt` file in this directory:

```bash
echo 'your_token' > token.txt
```

Or set an environment variable:

```bash
export SCRAPEDO_TOKEN='your_token'
```

## Security

The `token.txt` file is listed in `.gitignore`, so it will not end up in git.
