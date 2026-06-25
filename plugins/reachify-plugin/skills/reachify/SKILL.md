---
name: reachify
description: >-
  Run the reachify judgement-job worker loop. Each tick: poll the queue with
  `reachify get-job`; if it prints a path, spawn a sub-agent on that file, wait
  for it, then `reachify complete-job <path>`; if it prints nothing, there is no
  work — wait for the next tick. Use when asked to "run the reachify worker",
  "work the judgement-job queue", "poll reachify for jobs", "process reachify
  jobs", or to drive get-job / complete-job in a loop.
allowed-tools:
  - Bash(reachify *)
  - Read(/tmp/.reachify/**)
  - Write(/tmp/.reachify/**)
---

# Run reachify — the worker loop

`reachify` is a judgement-job worker for agents. `get-job` claims one job from
the backend and a self-contained agent file (`<work_dir>/<job-id>.md`).
Then the worker uses the file path and spawns off a sub agent via the Agent tool pointed to that path and
execute; the agent writes its judgement to a predefined output path; then
`complete-job` reads that output and reports it back.

You are using this skill **is** to become that worker. You as the worker: poll, spawn a sub-agent on the agent file via the **Agent tool**, and complete the job.

## The loop — one tick (the agent path)

Do exactly this each tick. `get-job` puts *only* the agent file path on stdout; all diagnostics go to stderr.

### 1. Poll for a job

```
Bash(reachify get-job)
```

Exit code is `0` whether or not a job was claimed.

Read the Bash tool results

- **None** → no claimable job this tick. **The tick is done — wait for the next
  one.** (stderr will say `None`)
- **A path** (e.g. `/tmp/.reachify/job-tone-001/job-tone-001.md`) → there is work.
  Continue. This is the **only** value you carry forward; remember it verbatim.
  Refered to has `file_path`

### 2. Spawn a sub-agent on the agent file — and wait

Call the **Agent tool** with the path from step 1. The agent file is self-contained. Sub agent should follow it precisely.

```
Agent({file_path})
```

Do **not** invent the output path or pass extra context — everything the agent
needs is in the file.

### 3. Complete the job — pass the same path

After the sub-agent finishes, report the result by handing `complete-job` the **exact path `get-job` printed**:

```
Bash(reachify complete-job {file_path})
```

On success stderr prints `completed job <id> (status=completed)` and stdout echoes the answer JSON.


## Gotchas

- **Only stdout carries the path.** `get-job` prints the path on stdout and
  everything else (claimed id, asset locations, expected output path) on stderr.
  Redirect them separately; never parse the path out of combined output.
- **Empty stdout is the "no job" signal, not an error.** Exit code is `0` either
  way. An empty line means wait for the next tick — do not retry in a tight loop.
- **Pass the path to `complete-job`, never a bare id.** The job id is recovered from the filename (`<job-id>.md` → `<job-id>`). Don't rename or move the agent file between `get-job` and `complete-job`, and don't strip the `.md`.
- **The sub-agent must actually write the output file.** `complete-job` reads the job's predefined `answer_path`; if the agent wrote nothing there, it fails with `No output found at <path>`. The write target is spelled out at the bottom of the agent file — that is what the sub-agent must obey.
- **One job per tick.** `get-job` claims a single job. To drain a full queue, run more ticks (e.g. `/loop`), not one giant pass.

## Troubleshooting

| Symptom | Cause / fix |
| --- | --- |
| `No profile found at ~/.reachify/.profile` | Run `reachify login` first. Ensure `HOME` is stable between login and the loop — a per-shell ephemeral `HOME` loses the profile. |
| `get-job` exits 0 but prints nothing, forever | Queue is empty or your filters match no jobs. Confirm the backend has claimable jobs; loosen/drop `--definition-key` etc. |
| `No output found at <path>` on `complete-job` | The sub-agent didn't write the answer file. Re-run the agent on the same agent file; check the file's trailing write-instruction. |
| `API error:` / 4xx from `complete-job` | You likely passed something other than the exact path `get-job` printed, or the lease expired. Re-`get-job` and pass the new path verbatim. |
| `No profile found` right after `login` | The profile lives at `~/.reachify/.profile`; verify `HOME` and that `login` reported `Saved profile for id=...`. |
