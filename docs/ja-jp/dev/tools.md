# Agnet�̖����������邽�߂̃c�[��

���̃y�[�W��Agent�̃v���t�@�C�����O�������������̂ɖ𗧂c�[���⎑�����ꗗ�ɂ��悤�Ǝv���܂��B

## pprof

����̐ݒ�ł́AAgent�� `5000` �ԃ|�[�g��pprof��HTTP�T�[�o�[��񋟂��Ă��܂��Bpprof�|�[�g����go�����^�C���̃v���t�@�C���iCPU�A�������Ȃǁj�⃉���^�C���Ɋւ����ʓI�ȏ����擾�ł��܂��B

pprof�̃h�L�������g: https://golang.org/pkg/net/http/pprof/

�w�肵����ǉ������肷��ɂ́A���̃R�}���h������͂��Ă�������:

* �S���[�`���ꗗ:
```sh
curl http://localhost:5000/debug/pprof/goroutine?debug=2
```
* go �q�[�v�̃v���t�@�C��:
```sh
go tool pprof http://localhost:5000/debug/pprof/heap
```

## expvar

����̐ݒ�ł́AAgent�� `5000` �ԃ|�[�g��HTTP�T�[�o�[�o�R��expvar�ϐ���JSON�`���Ŏ擾�ł��܂��B

expvar�̃h�L�������g: https://golang.org/pkg/expvar/

�i���ꂼ��̃L�[�z���́jAgent�̕ϐ����o�͂��܂��B����̐ݒ�ł́Aexpvar�́A`runtime.Memstats`�����ʓI�ȃ��������v�����o�͂��܂��B([`runtime.MemStats docs`][runtime-docs]���Q�Ƃ��Ă��������B). ���ɁA `Sys` �� `HeapSys` �A `HeapInuse` �ϐ��͋����[���ł��B

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
