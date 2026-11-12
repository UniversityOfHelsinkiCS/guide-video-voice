# guide-video-voice

The voice service that narrates the user guide videos. The voice is [Chatterbox Multilingual](https://github.com/resemble-ai/chatterbox) (MIT) on CPU, with pinned weights and the voices baked into the image. It speaks `fi`, `sv` and `en`, and watermarks every clip inaudibly.

## Using it

In an app's Compose file:

```yaml
tts:
  image: registry.version.helsinki.fi/toska/openshift/kontit/guide-video-voice:<version>
  profiles: [manual]
  read_only: true
  tmpfs: [/tmp]
  cap_drop: [ALL]
  security_opt: [no-new-privileges:true]
  ports:
    - 127.0.0.1:5005:5005
```

Pulling the image needs `docker login registry.version.helsinki.fi`.

- `GET /health` returns `{ version, languages, voices }`.
- `POST /synthesize` takes `{ text, lang, voice? }` and returns a WAV. Errors are `{ error }`, with status 400 for a bad request.
- Generation is seeded, so the same request gives the same audio. Apps cache their clips by the image version, voice, language and text.

## Releasing

Publish a GitHub release with a new `v<version>` tag on master. CI pushes the image as `guide-video-voice:<version>` and never replaces an existing version. Pushes to master only build and test it. Any change to the image changes the audio, so apps regenerate their clips when they switch to a new version.

- The voice's settings are in `GENERATE_OPTIONS` in `server.py`.
- To update the Python packages, run `uv lock --upgrade`. The direct dependencies are pinned in `pyproject.toml`.
- Custom voices and their consent are in [voices/README.md](voices/README.md).

## Locally

```bash
docker build --build-arg VERSION=dev -t guide-video-voice:dev .
docker run --rm -p 127.0.0.1:5005:5005 guide-video-voice:dev
```
