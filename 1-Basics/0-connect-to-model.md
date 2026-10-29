Assuming you mean Nous Research's Hermes Agent, you set it up as a `custom` OpenAI-compatible provider. Your Python snippet maps directly to three settings: `base_url`, `api_key`, and the model name.

## Option 1: the wizard (easiest)

```bash
hermes model
# choose: "Custom endpoint"
```

Enter the base URL, API key, and model name when it asks.

## Option 2: edit `~/.hermes/config.yaml`

```yaml
model:
  provider: custom
  base_url: https://backend.sovereigneg.com/v1
  default: glm-5.3
  api_key: YOUR_API_KEY
```

`base_url` must end in `/v1`, as yours does. `provider: custom` is required for any endpoint that isn't built in.

## Option 3: keep the key out of the YAML (recommended)

Put the key in `~/.hermes/.env`:

```bash
OPENAI_API_KEY=YOUR_API_KEY
```

If `api_key` is omitted from the YAML, Hermes falls back to `OPENAI_API_KEY`. Then the YAML only needs:

```yaml
model:
  provider: custom
  base_url: https://backend.sovereigneg.com/v1
  default: glm-5.3
```

You can also set everything from the CLI. API keys go to `.env`, and everything else goes to `config.yaml`:

```bash
hermes config set model.provider custom
hermes config set model.base_url https://backend.sovereigneg.com/v1
hermes config set model.default glm-5.3
hermes config set OPENAI_API_KEY YOUR_API_KEY
```

Restart Hermes and open a new session, then send a short prompt to test.

## Notes

- Don't use `OPENROUTER_API_KEY` for this. Hermes deliberately keeps that key out of requests to non-OpenRouter servers.
- Hermes needs a model context window of roughly 64k or more to work properly as an agent. Check that `glm-5.3` on that endpoint offers that and supports tool calling.
- If you want to switch between several endpoints, define named providers in `config.yaml` and select one with `provider: "custom:<name>"`. In a session you can use `/model custom:<name>:<model>`.
- If you get "connection refused" or auth errors, check that the YAML indentation is valid and that the key has no extra spaces or quotes.

Field names can differ slightly between Hermes versions, so if something doesn't match, run `hermes model` and let the wizard write the config for you.