# Playground Builder: instructions for the agent

## What this is

The task is to turn a skill or an API into a standalone browser-based playground.
The user says `Make a playground for [skill], instructions [link]`,
and the agent figures out the scripts itself, asks questions and builds an interactive UI.

## User prompt

```
/playground Make a playground for [skill name]
```

That's all. The agent does the rest.

---

## Instructions for the agent

### Step 1: Study the skill

Read the skill's scripts.
Find:
- All API endpoints (URL, method, parameters, response format)
- All command-line parameters (--prompt, --resolution, etc.)
- Whether API keys are used (config/.env)
- Input data format (text, files, URL)
- Output data format (image, text, JSON, file)
- Whether there is queue/polling (submit → poll status → get result)

### Step 2: Ask the user

Use AskUserQuestionTool to ask questions.
Ask only relevant ones: don't ask about files if the skill is text-only.

Possible questions:
- **Mode**: if the skill supports several (generate/edit, analyze/export)
- **UI language**: Russian or English
- **Results folder**: where to save
- **Iterations**: whether a result should be reusable as an input
- **Extra features**: drawing regions, templates, presets

### Step 3: Build the playground

Build the playground as a set of files in the folder the user specified.

---

## Cases: what to do when

### Case: The script has an API key

If the script loads from `.env` (the `KEY=...` pattern):
- Add a text field to the UI for pasting the key
- Add a file picker for loading the .env file
  - Remove the `accept` attribute (`.env` has no MIME type, the picker would hide it)
  - In the hint, give the path depending on where the skill lives, for example: `~/.claude/plugins/cache/[marketplace]/[skill]/[version]/skills/[skill]/config`
  - Add a hint: "Cmd+Shift+G to jump to a path"
- Parsing: `text.match(/KEY_NAME\s*=\s*(.+)/)`
- Keep the key only in `state` (tab memory), not in localStorage

### Case: The API does not support CORS

Almost no API returns `Access-Control-Allow-Origin` for localhost.
Even `file://` and `http://localhost` will be blocked.

Create next to the HTML:

**server.py**: a proxy on stdlib Python:
- `http.server.SimpleHTTPRequestHandler` for static files
- `do_GET/do_POST/do_PUT`: if the path is `/api/*`, proxy to the real API
- `do_OPTIONS`: return 200 + CORS headers for the preflight
- Headers: forward `Authorization`, and `Content-Type` only for POST/PUT
- GET requests: do NOT send Content-Type (some APIs answer 405)
- `urllib.error.HTTPError`: proxy the code and the body, don't swallow it
- Logging: `print(f"[proxy] {method} {target}")` for debugging

**start.sh**:
```sh
#!/bin/sh
PORT=8090
echo "Playground: http://localhost:$PORT/[name].html"
open "http://localhost:$PORT/[name].html"
cd "$(dirname "$0")" && uv run server.py $PORT
```

### Case: Presigned URL upload (two-step upload)

Pattern: POST initiate → get `{upload_url, file_url}` → PUT the file to `upload_url`.

A presigned URL contains query parameters (`?signature=...&token=...`).

Proxying:
- In HTML: `fetch('/api/put?url=' + encodeURIComponent(presignedUrl), {method: 'PUT', body: file})`
- In server.py: parse with `urllib.parse.parse_qs`, NOT with `split("?url=")`

```python
elif path.startswith('/api/put?'):
    qs = urllib.parse.urlparse(path).query
    target = urllib.parse.parse_qs(qs).get('url', [''])[0]
```

### Case: Queue-based API (long polling)

Pattern: submit → get request_id → poll status → get result.

**IMPORTANT**: the URLs for submit and for status/result may differ.
Always check all the URLs in the script, and don't assume a single base path.

In HTML:
- A progress bar with animation
- Polling at a reasonable interval (2-5 sec)
- A timeout
- Handling of all statuses from the script (success, fail, in progress, queued, etc.)
- On completion, fetch the result (the endpoint may differ from status)

In server.py: separate routes for submit and status if the URLs differ.

### Case: The skill works with files (images, documents)

- A drag & drop zone with thumbnail previews
- `<input type="file">` as a fallback
- For images: show them on a `<canvas>`
- For the canvas: let the user draw rectangular regions (mousedown → mousemove → mouseup)
- Each region is a text note in the sidebar
- The notes go into the prompt with coordinates as a percentage of the image size

Loading a default image (if the file sits next to the HTML):
- Use `canvas.toBlob()` to convert to a File, NOT `fetch()`
- Wrap in `try/catch`, because from `file://` the canvas will be tainted

### Case: The result can be iterated on

If the result is the same type as the input (image → image):
- Save the URL/blob of every result in a `state.history` array
- Show thumbnails in the sidebar
- Clicking a thumbnail loads it as the new source
- A "Restore original" button
- After generation: update `state.imageFile` from the result for the next iteration

### Case: AI feedback through the Claude CLI

If the result benefits from analysis or iteration with AI:
- Add an `/api/claude` endpoint to server.py
- The server calls `claude -p "prompt" --output-format json` through `subprocess.run()`
- To keep context: `claude --resume SESSION_ID -p "prompt"`
- In HTML: an "Ask Claude" button sends the result + a question to `/api/claude`

### Case: Interface language

Always ask through AskUserQuestion. Don't guess from the language of the prompt.

If the user chose Russian:
- All UI elements: headings, labels, placeholders, hints, buttons, statuses, errors
- Buttons: "Запустить" (Run), "Скопировать" (Copy), "Очистить все" (Clear all), "Скачать" (Download), "Вернуть оригинал" (Restore original)
- Statuses: "Загружаю..." (Loading...), "Генерация... (обычно 15-30 сек)" (Generating... usually 15-30 sec), "Готово!" (Done!), "Ошибка: ..." (Error: ...)

---

## Playground architecture

### Files

```
output-folder/
  [name]-playground.html   : the UI, a single file, inline CSS + JS
  server.py                : proxy (if needed)
  start.sh                 : launcher (if a proxy is needed)
```

### UI requirements

- A single HTML file, inline CSS + JS
- Dark theme
- Run button: `disabled` until the required fields are filled
- Progress/status: the user must see what is happening
- Choose the layout to fit the skill's task (sidebar + preview, full screen, etc.)

---

## Pitfalls (must read)

1. **file:// does not work**. `fetch()`, `canvas.toBlob()` for cross-origin,
   reading files by path: all blocked. Always go through localhost.

2. **CORS blocks even localhost**. A proxy is almost always needed.

3. **`.env` has no MIME type**. The file picker will hide it.
   Remove the `accept` attribute from `<input type="file">`.

4. **A presigned URL contains `?`**. It can't be parsed with a simple split.

5. **Submit and status on different URLs**. Don't assume a single base path.

6. **GET + Content-Type = 405**. Some APIs reject a GET with a Content-Type.

---

## Example: what the dialogue looks like

**User:**
```
/playground Make an interface for [skill name]
```

**Agent** (after studying the scripts): asks questions through AskUserQuestionTool,
only those relevant to this particular skill.

**User:** answers the questions.

**Agent:** Creates the files, starts it, and the playground works.
