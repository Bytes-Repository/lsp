# lsp — serwery Language Server Protocol w H#

Biblioteka dla [H#](https://github.com/Bytes-Repository) instalowana przez **bytes**.
Daje wszystko, czego potrzebuje serwer LSP: transport JSON-RPC po stdio,
parser JSON, cykl zycia serwera, magazyn dokumentow z synchronizacja
inkrementalna, konwersje pozycji UTF-16 i gotowe budowniczowie obiektow
protokolu (diagnostyki, hover, uzupelnianie, symbole, edycje...).

## Instalacja

```bash
bytes add lsp
```

W kodzie (alias **musi** nazywac sie `lsp`):

```
use "bytes -> lsp" from "lsp"
```

## Minimalny serwer

```
use "bytes -> lsp" from "lsp"

fn main() is
    let mut srv: lsp::Server = lsp::new_server("moj-ls", "0.1.0")
    srv = lsp::enable(srv, "hoverProvider")

    while !lsp::finished(srv) is
        srv = lsp::next(srv)
        match lsp::method(srv) is
            "textDocument/hover" => is
                lsp::reply(srv, lsp::proto::hover("**" + lsp::cursor_word(srv) + "**"))
            end
            "" => is end
            _ => is srv = lsp::unhandled(srv) end
        end
    end
    exit(lsp::exit_code(srv))
end
```

`lsp::next` obsluguje sam `initialize`, `shutdown`, `exit`, `$/cancelRequest`
i bledy protokolu oraz aktualizuje dokumenty przy `didOpen/didChange/didClose`
(dopiero potem oddaje ci wiadomosc). Ty piszesz tylko `match` po metodzie.
Galaz `_ => srv = lsp::unhandled(srv)` odpowiada `MethodNotFound` na nieznane
zadania. Galaz `""` to koniec strumienia (EOF).

Pelny przyklad z diagnostykami, hover, uzupelnianiem i symbolami:
[`examples/hello-ls`](examples/hello-ls/src/main.h#).

## Moduly

| Sciezka | Co robi |
|---|---|
| `lsp::...` | `Server`, petla, dostep do wiadomosci, dokumentow i kursora, wysylanie |
| `lsp::json::...` | JSON: `get`, `get_str`, `get_int`, `items`, `keys`, `put`, `quote`, `array`... |
| `lsp::text::...` | pozycje i edycje, `word_at`, `uri_to_path`, `path_to_uri` |
| `lsp::proto::...` | `range`, `diagnostic`, `hover`, `completion_item`, `document_symbol`, `workspace_edit`... |
| `lsp::rpc::...` | `read_message`, `send`, budowanie odpowiedzi/bledow/powiadomien |
| `lsp::docs::...` | magazyn dokumentow (`open`, `apply_changes`, `text`, `line`...) |

Stale (`LSP_SEV_ERROR`, `LSP_KIND_KEYWORD`, `LSP_SYM_FUNCTION`, `LSP_ERR_*`...)
sa globalne — kompilator nie prefiksuje stalych. Dokladny opis: [docs/API.md](docs/API.md).

## Konwencje

* **Wartosc JSON = string z jej surowym zapisem.** `lsp::json::get(raw, "params.textDocument.uri")`
  zwraca surowy fragment; `get_str`/`get_int`/`get_bool` dekoduja. Zle sciezki
  daja `""`/`0`/`false`, nigdy panike.
* **Stan jest funkcyjny.** Funkcje zmieniajace `Server` zwracaja nowy `Server`:
  `srv = lsp::enable(srv, "hoverProvider")`. Funkcje wysylajace (`reply`,
  `notify`, `publish_diagnostics`, `log_info`) nic nie zwracaja.
* **stdout nalezy do protokolu.** Nie uzywaj `write()` w serwerze. Logi:
  `lsp::log_info(srv, "...")` (panel Output edytora).
* **Pozycje** sa liczone w jednostkach z `lsp::encoding(srv)` (domyslnie UTF-16).
  `lsp::prefer_utf8(srv)` negocjuje UTF-8, jesli klient to obsluguje.

## Ograniczenia H#, o ktorych warto wiedziec

* H# nie ma GC — biblioteka minimalizuje alokacje, ale dlugo dzialajacy serwer
  bedzie rosnac. Dlatego domyslna synchronizacja jest **inkrementalna**
  (`lsp::set_sync(srv, LSP_SYNC_FULL)` zwieksza zuzycie pamieci przy duzych plikach).
* Interpreter H# czyta bajty > 0x7F jako Latin-1; polskie znaki dzialaja poprawnie
  dopiero w programie **skompilowanym** (`bytes build`).
* Nie ma jeszcze zapisu na stderr — logi ida przez `window/logMessage`.

## Testy

```bash
bytes test                                                  # testy jednostkowe (src/lsp_test.h#)
bytes build && python3 tests/e2e_client.py build/hello-ls   # test end-to-end (w przykladzie)
```

`tests/e2e_client.py` to prosty klient LSP w Pythonie, ktory uruchamia serwer,
wysyla `initialize`, `didOpen`, `didChange`, `hover`, `completion`,
`documentSymbol`, `shutdown`, `exit` i sprawdza odpowiedzi.

## Licencja

MIT
