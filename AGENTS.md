# Invoice — Agent Instructions

CLI tool that generates PDF invoices. Single `main` package, ~4 source files.

## Build & Run

```bash
go build .               # build binary
go run . generate --help # run without building
go install .             # install to $GOPATH/bin
```

No test suite exists. Validate changes by running the binary with various flag combinations and inspecting the generated `invoice.pdf`.

```bash
go run . generate --from "Me" --to "Client" --item "Work" --quantity 1 --rate 100
open invoice.pdf
```

## File Map

| File | Purpose |
|------|---------|
| `main.go` | `Invoice` struct, cobra CLI, `generateCmd` runner |
| `pdf.go` | All PDF rendering (`writeLogo`, `writeTitle`, `writeRow`, `writeTotals`, etc.) |
| `import.go` | JSON/YAML config file import; flag override logic |
| `currency.go` | `currencySymbols` map (currency code → symbol string) |

## Key Architecture Facts

- **PDF layout uses absolute pixel offsets** — column positions are constants at the top of `pdf.go` (`quantityColumnOffset`, `rateColumnOffset`, `amountColumnOffset`). Adjust these when changing layout.
- **Fonts are embedded** via `//go:embed` in `main.go`. The `Inter/` directory must stay in the repo. Do not move or rename font files.
- **`file` is a package-level global** (`var file = Invoice{}`). The cobra flag set mutates it directly via `StringVar`, `IntSliceVar`, etc. in `init()`.
- **Flag precedence**: import file loads first, then explicit CLI flags override individual fields (see `importData` in `import.go`).
- **Multi-line `--from`/`--to`/`--note`** supported via `\n` literals in the string (replaced in `pdf.go` `write*` functions).
- **Items, quantities, and rates are parallel slices** — they are aligned by index. Mismatched lengths are handled gracefully (defaults to 1/0.0 for missing entries).

## Configuration

Three ways to supply data (in precedence order, highest last):

1. Environment variables: `INVOICE_FROM`, `INVOICE_TO`, `INVOICE_TAX`, `INVOICE_RATE`, `INVOICE_LOGO`
2. Import file (`--import path.json` or `.yaml`)
3. CLI flags (override import values)

## Releases

Releases are published via [goreleaser](https://goreleaser.com) on tag push (see `.github/workflows/release.yml`). To cut a release, push a semver tag.

## Adding a Currency

Add an entry to `currencySymbols` in `currency.go`. The `--currency` flag accepts the key; the value is prepended to all monetary amounts.
