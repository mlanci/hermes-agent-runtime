# Railway deploy branch

This branch (`railway`) is upstream `NousResearch/hermes-agent` `main` plus the
minimum needed to build and run on Railway. It is the deploy source for Mike's
Hermes runtimes; the operating docs live in `mlanci/hermes-scaffolding`
(`RUNTIMES.md`, `runtime-bootstrap/`).

## Patches (all in `Dockerfile` + `railway.toml`)

| Change | Why |
|---|---|
| `VOLUME [ "/opt/data" ]` removed | Railway's Dockerfile builder rejects `VOLUME`; the volume is attached Railway-side at `/opt/data`. |
| `RUN chmod 0755 /opt/hermes/docker/*.sh` + s6 `run`/`finish` scripts | `railway up` from Windows uploads a tarball with no exec bits; upstream's `COPY --chmod=a+rX` only keeps bits that already exist. |
| `CMD [ "gateway", "run" ]` | Upstream `CMD []` launches the interactive CLI. Railway has no TTY; we want the s6-supervised gateway. |
| `--extra voice --extra tts-premium` baked into the venv | Telegram voice notes (faster-whisper STT) and ElevenLabs TTS must survive image rebuilds instead of relying on lazy install. |
| `railway.toml` | Pins the Dockerfile builder and restart policy. No `startCommand`, so the image ENTRYPOINT/CMD run as PID 1 and s6 supervision works. |

## Updating from upstream

```bash
git fetch upstream
git rebase upstream/main          # or: git merge upstream/main
# resolve conflicts (usually only the Dockerfile extras line), then:
git push --force-with-lease origin railway
railway up                        # from a checkout linked to the target project
```
