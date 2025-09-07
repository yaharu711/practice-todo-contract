# todo-contract

Single Source of Truth for the Todo API (OpenAPI 3.0).  
CI runs **Spectral** lint; PRs fail if the spec violates rules.

## Raw URLs (pin these in clients)

- Latest on `main`  
  `https://raw.githubusercontent.com/yaharu711/todo-contract/main/openapi/todo.yaml`

- Tag-pinned (recommended for releases)  
  `https://raw.githubusercontent.com/yaharu711/todo-contract/v1.0.0/openapi/todo.yaml`

## Usage (examples)

### Frontend (curl → codegen)

```bash
curl -fsSL https://raw.githubusercontent.com/yaharu711/todo-contract/main/openapi/todo.yaml \
  -o spec/openapi.yaml
# then run your codegen, e.g.:
# openapi-typescript spec/openapi.yaml --output src/api/client/schema.d.ts
# orval --config orval.config.ts
```

### Backend (curl → contract tests)

```bash
curl -fsSL https://raw.githubusercontent.com/yaharu711/todo-contract/main/openapi/todo.yaml \
  -o storage/app/spec/openapi.yaml
# then run PHPUnit with OpenAPI validation
```

## Notes

- Keep it lightweight: only Spectral lint here.
- Breaking-change detection is handled in client repos (CI).
- If you need stability, **pin clients to a tag** instead of `main`.

## 環境は Docker を使う

原則：リポ直下に Dockerfile と docker-compose.yml を置き、作業は全部コンテナ内で（lint/codegen/dev）。

**node_modules は“名前付きボリューム”**を使う（ホストの他プロジェクトと干渉しない・権限問題も回避）。

### 使い方

# 一度ビルドしておけば OK
docker compose up --build # PR/Push 前のローカル lint もこれで
docker compose run --rm contract npm run lint

これで他ディレクトリの Node 環境とは完全分離。Spectral は CI でも同じ Node 20 で走ります。
