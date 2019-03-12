# Agnetの問題解決をするためのツール

このページはAgentのプロファイリングや問題解決をするのに役立つツールや資料を一覧にしようと思います。

## pprof

既定の設定では、Agentは `5000` 番ポートでpprofのHTTPサーバーを提供しています。pprofポートからgoランタイムのプロファイル（CPU、メモリなど）やランタイムに関する一般的な情報を取得できます。

pprofのドキュメント: https://golang.org/pkg/net/http/pprof/

指定したり追加したりするには、次のコマンドを手入力してください:

* ゴルーチン一覧:
```sh
curl http://localhost:5000/debug/pprof/goroutine?debug=2
```
* go ヒープのプロファイル:
```sh
go tool pprof http://localhost:5000/debug/pprof/heap
```

## expvar

既定の設定では、Agentは `5000` 番ポートのHTTPサーバー経由でexpvar変数をJSON形式で取得できます。

expvarのドキュメント: https://golang.org/pkg/expvar/

（それぞれのキー配下の）Agentの変数を出力します。既定の設定では、expvarは、`runtime.Memstats`から一般的なメモリ統計情報を出力します。([`runtime.MemStats docs`][runtime-docs]を参照してください。). 特に、 `Sys` と `HeapSys` 、 `HeapInuse` 変数は興味深いです。

Using the `jq` command-line tool, it's rather easy to explore and find relevant variables, for example:
```sh
# Find total bytes of memory obtained from the OS by the go runtime
curl -s http://localhost:5000/debug/vars | jq '.memstats.Sys'
# Get names of checks that the collector's check runner has run
curl -s http://localhost:5000/debug/vars | jq '.runner.Checks | keys'
```

## delve

A debugger for Go.

[Project page][delve-project-page]

Example usage:
```sh
$ sudo dlv attach `pgrep -f '/opt/datadog-agent/bin/agent/agent run'`
(dlv) help # help on all commands
(dlv) goroutines # list goroutines
(dlv) threads # list threads
(dlv) goroutine <number> # switch to goroutine
```

## gdb

GDB can in some rare cases be useful to troubleshoot the embedded python interpreter.
See https://wiki.python.org/moin/DebuggingWithGdb

Example usage (using the legacy `pystack` macro):
```sh
sudo ./gdb --pid <pid>
info threads
thread <number> # switch to thread
pystack # python stacktrace of current thread
```

For simple debugging cases, you can simply use the python-provided `pdb` to jump into
a debugging shell by adding to the python code that's run:
```python
import pdb
pdb.set_trace()
```
and running the agent in the foreground.

[runtime-docs]: https://golang.org/pkg/runtime/#MemStats
[delve-project-page]: https://github.com/derekparker/delve
