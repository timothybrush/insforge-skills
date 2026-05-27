# Video Generation

Use OpenRouter video models for asynchronous generation jobs. Keep this guide
high level: rely on OpenRouter docs for request/response details, and use
InsForge for job tracking, scheduled polling, Storage, and frontend status.

Official OpenRouter references:

- [Video generation](https://openrouter.ai/docs/guides/overview/multimodal/video-generation)
- [Submit a video generation request](https://openrouter.ai/docs/api/api-reference/video-generation/create-videos)
- [Poll video generation status](https://openrouter.ai/docs/api/api-reference/video-generation/get-videos)
- [List video models](https://openrouter.ai/docs/api/api-reference/video-generation/list-videos-models)
- [Models API](https://openrouter.ai/docs/api-reference/models/get-models)

## Recommended InsForge Flow

1. Start the OpenRouter video job from server-side code only.
2. Store a `video_jobs` row in InsForge with the requesting user, prompt, model,
   OpenRouter job ID, polling URL, status, and next poll time.
3. Use one InsForge Edge Function as the poller.
4. Trigger that poller with an InsForge Schedule, for example every 30 seconds.
5. In each scheduled run, inspect due unfinished jobs until one is completed.
6. For the first completed job found, download the video server-side with
   `OPENROUTER_API_KEY`, upload it to InsForge Storage, save `video_url` and
   `video_key`, then stop. Leave the rest for later schedule ticks.
7. Let the frontend read `video_jobs` from InsForge or subscribe via Realtime.
   The browser should never poll OpenRouter or see `OPENROUTER_API_KEY`.

Use a DB **lease** for in-flight work, not a permanent lock. `lease_owner` plus
`lease_expires_at` communicates that a crashed poller can be retried after the
lease expires.

## Deno Memory

Downloading a video in an Edge Function can buffer the entire MP4 in memory,
depending on the upload path. That can be fine for short, low-resolution clips
when you process only one video per schedule run. For larger videos, use a
Compute worker or a streaming/multipart upload path instead of buffering the
whole file inside Deno.

## Best Practices

1. Use server-side polling as the default baseline; do not pass `callback_url`
   unless you are ready to verify webhook signatures.
2. Keep each scheduled invocation bounded: poll a few due jobs, process at most
   one completed video, then return.
3. Store generated videos in InsForge Storage; save both `url` and `key` in the
   database.
4. Store prompt, model, duration, resolution, status, polling URL, errors, retry
   metadata, and Storage references for support/debugging.
5. Use OpenRouter model docs for exact parameters and supported durations,
   sizes, and input formats.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Treating video generation as synchronous | Store a job row and poll from a scheduled server-side function |
| Letting one cron invocation drain the whole queue | Process at most one completed video per run |
| Polling OpenRouter from the browser | Poll from an InsForge Edge Function or server worker |
| Saving only the OpenRouter content URL | Download server-side, upload to Storage, save `url` and `key` |
| Assuming `unsigned_urls[0]` is a public app URL | Treat it as a server-side download URL and send Authorization |
| Buffering large videos in Deno Edge Functions | Use short clips or move large outputs to Compute/streaming upload |
