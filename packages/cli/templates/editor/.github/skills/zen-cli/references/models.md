# Models - `zen models`

```
zen models <providers|ls|search|show|test|browse> [ref] [options]
```

Alias: `model`.

Answers **what can I use**. `zen check` answers the other question - whether one
particular project works - and needs a project to do it. This one needs nothing
but a credential.

## Subcommands

| Command                     | What it does                                           |
| --------------------------- | ------------------------------------------------------ |
| `zen models`                | Providers, credential source, how many models, how old |
| `zen models <provider>`     | Short for `ls <provider>`                              |
| `zen models ls [provider]`  | Everything it serves                                   |
| `zen models search <query>` | Narrow it                                              |
| `zen models show <ref>`     | One model, every field the vendor gave                 |
| `zen models test <ref> …`   | One real minimal call per ref, and a verdict           |
| `zen models browse`         | Interactive TUI for a person; needs a terminal         |

Filters, on `ls` and `search`: `--chat`, `--embeddings`, `--images`, `--audio`,
`--tools`, `--vision`, `--free`, `--min-context <n>`, `--provider <name>`,
`--limit <n>`, `--all`, `--refresh`.

`browse` is for a person at a terminal and refuses without one (exit 2). An
agent wants `ls --json` or `search` instead.

## Where the lists come from

The providers themselves, cached for a day in `~/.zenera/neo/catalog`. When a
provider cannot be asked, the last listing is used and reported as stale; only
if there was never one does a short built-in list stand in. Every row says which
it was - `--json` carries `source` per model and `origin` per provider.

`--refresh` bypasses the cache. It is the only thing that does.

`zen models` on its own does **not** go to the network. `ls` and `search` do.

## Testing a model

```
zen models test openai:gpt-4o-mini
zen models test vertex:gemini-embedding-001 --embedding
```

The role is taken from the flag, else from what the provider says the model is
for, else from the id. One minimal call: `ok` for a chat model, one short vector
for an embedder - the embedding's width is reported, which matters because a
model serving a different number of dimensions is not interchangeable with the
one an index was built on.

Four verdicts, and they want four different actions:

| Verdict     | Means                                        | Do                         |
| ----------- | -------------------------------------------- | -------------------------- |
| `answers`   | it works                                     | nothing                    |
| `refused`   | the credential was rejected                  | `zen key check <provider>` |
| `blocked`   | the credential was fine, the account said no | the `fix` printed under it |
| `no answer` | it could not be reached                      | try again                  |

Exit `0` when every ref answered, `4` when any did not, `2` for a ref that does
not parse.

## Recovering from a blocked model

This is what the command is for.

```
$ zen models test vertex:gemini-embedding-001
vertex:gemini-embedding-001  blocked  Vertex AI API has not been used in project my-proj …
fix vertex:gemini-embedding-001 gcloud services enable aiplatform.googleapis.com --project my-proj
error 1 of 1 did not answer
        see what else there is: zen models ls --embeddings
```

Two ways out. Run the `gcloud` line, or take a different model: list what the
account serves, test the one you want, and put its ref where the old one was.

```
$ zen models ls --embeddings
$ zen models test openai:text-embedding-3-small
openai:text-embedding-3-small  answers 412ms  1536 dims
```

Test one model, not the whole list - each test is a billable call.

## What it will not do

There is no `test --all`. A sweep across every model on the machine is a bill,
not a diagnostic - name the refs you care about, or use `pick`.

There is no ranking. The candidate order in `pick` is cheapest-and-fastest
first, which is about how quickly an answer arrives, not about which model is
better.
