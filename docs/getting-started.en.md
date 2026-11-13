# How to try it

[Versión en español](getting-started.md)

## Walkthrough 1: the router (nothing to install)

Requirements: Python 3.9 or newer.

```bash
git clone <URL of this repository>
cd ai-split-raspberry-pi
python3 -m unittest discover -s tests -v
```

Expected result: 22 green tests (11 for the router, 10 for the monitor and 1 for the bilingual documentation). The tasks in `demo/fixtures/tasks.json` were invented for this repository.

To explore:
- Change a label in `demo/fixtures/tasks.json` and run the tests: the router should send the task to a different participant.
- Try `route(Task("x", frozenset()), usage_pct=90)` in a Python console: an unclassified task with the reserve
  used up returns `queue`.

## Walkthrough 2: the delegation script

Requirements: Bash.

```bash
bash tests/test_delegate.sh
```

Expected result: 5 "ok" cases (complete answer, busy, permission prompt, file without end marker, timeout). It uses a simulated `tmux`.

To use it with a real tool you also need `tmux` and a command-line AI running in a session:

```bash
IDLE_PATTERN='text shown by the empty input box' \
OUT_DIR=/absolute/path/for/answers \
src/delegate.sh <tmux_session> "one-line request" 600
```

- `IDLE_PATTERN`: the text the tool shows while it waits for a request; it changes from one tool to another.
- `OUT_DIR`: where the answers are written (by default `./responses`, converted to an absolute path).
- Limits: it is a reference; detecting "idle" and "asking for permission" is heuristic. See [`limitations.md`](limitations.md) (Spanish).
