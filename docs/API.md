# API biblioteki lsp

Wywolania: `lsp::f(...)` (lib), `lsp::json::f`, `lsp::text::f`, `lsp::proto::f`,
`lsp::rpc::f`, `lsp::docs::f`. Alias importu musi byc `lsp`.

## `lsp::` — serwer

| Funkcja | Opis |
|---|---|
| `new_server(name, version) -> Server` | nowy serwer (sync inkrementalny, UTF-16) |
| `set_capability(srv, key, raw) -> Server` | dowolna zdolnosc: `set_capability(srv, "hoverProvider", "true")` |
| `enable(srv, key) -> Server` | zdolnosc boolean `true` |
| `set_sync(srv, kind) -> Server` | `LSP_SYNC_NONE/FULL/INCREMENTAL` |
| `prefer_utf8(srv) -> Server` | negocjuj pozycje w UTF-8 |
| `next(srv) -> Server` | wczytaj kolejna wiadomosc (obsluguje cykl zycia i dokumenty) |
| `finished(srv) -> bool`, `exit_code(srv) -> int` | koniec petli / kod dla `exit()` |
| `unhandled(srv) -> Server` | `MethodNotFound` dla zadan, nic dla powiadomien |

Biezaca wiadomosc: `method`, `raw`, `id`, `params`, `param(srv, path)`,
`param_str`, `param_int`, `param_bool`, `uri`, `pos_line`, `pos_char`,
`is_request`, `is_notification`, `is_response`.

Inicjalizacja: `root_uri`, `root_path`, `client_capabilities`,
`initialize_params`, `encoding`.

Dokumenty: `text(srv, uri)`, `lines(srv, uri)`, `doc_line(srv, uri, n)`,
`is_open`, `open_uris`, `docs(srv)`.
Kursor: `cursor_line`, `cursor_byte`, `cursor_prefix`, `cursor_word`, `cursor_word_range`.

Wysylanie: `reply(srv, result)`, `reply_error(srv, code, msg)`,
`notify(srv, method, params)`, `request(srv, method, params) -> Server`,
`publish_diagnostics(srv, uri, diags)`, `clear_diagnostics(srv, uri)`,
`log_info/log_warn/log_error(srv, msg)`, `show_info/show_warn/show_error(srv, msg)`.

Wiadomosc, ktora `next` zwraca po `didOpen/didChange/didClose`, ma juz
zaktualizowany magazyn dokumentow. Odpowiedzi klienta na `request` przychodza
jako `is_response(srv)` (metoda `""`, ale `id(srv) != ""`).

## `lsp::json::`

`get(raw, path)`, `get_str`, `get_int`, `get_bool`, `has`, `kind`, `count`,
`items`, `item`, `keys`, `valid`, `to_str`, `to_int`, `to_bool`, `unescape`;
budowanie: `quote/str`, `num`, `boolean`, `null_value`, `empty_object`,
`empty_array`, `array`, `array_of_strings`, `array_of_ints`, `object_of`,
`put`, `put_str`, `put_int`, `put_bool`, `append`.
Sciezki: `a.b.0.c` (segment cyfrowy = indeks tablicy).

## `lsp::text::`

`split_lines`, `join_lines`, `col_to_byte(line, col, enc)`, `byte_to_col`,
`line_length`, `offset_at(lines, line, col, enc)`, `line_of_offset`,
`col_of_offset`, `apply_edit(lines, sl, sc, el, ec, text, enc)`,
`word_at(line, byteoff)`, `word_start`, `word_end`, `is_ident_byte`,
`uri_to_path`, `path_to_uri`, `pct_decode`, `pct_encode`.

## `lsp::proto::`

`position`, `range`, `location`, `text_edit`, `markup/markdown/plaintext`,
`diagnostic`, `diagnostic_code`, `publish_diagnostics_params`, `hover`,
`hover_range`, `completion_item`, `completion_insert`, `completion_doc`,
`completion_list`, `parameter`, `signature`, `signature_help`,
`document_symbol`, `symbol_information`, `workspace_edit`, `command`,
`code_action`, `folding_range`, `semantic_tokens`, `completion_options`,
`signature_help_options`, `rename_options`, `semantic_tokens_options`.

Stale: `LSP_SEV_*`, `LSP_SYNC_*`, `LSP_KIND_*` (CompletionItemKind),
`LSP_SYM_*` (SymbolKind), `LSP_MSG_*`, `LSP_ERR_*`.

## `lsp::rpc::`

`read_message()`, `send(body)`, `response`, `error_response`, `notification`,
`request`, `msg_method`, `msg_id`, `msg_params`, `is_request`,
`is_notification`, `is_response`, `log_message`, `show_message`.

## `lsp::docs::`

`new_store`, `open`, `close`, `set_text`, `apply_changes`, `text`, `lines`,
`line`, `line_count`, `version`, `language`, `uris`, `count`, `is_open`, `index_of`.
