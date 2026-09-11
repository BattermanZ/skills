---
name: clip-web-article
description: Use when asked to save, clip, transcribe, or archive a web article (news story, blog post, etc.) into the vault — especially when the article has images that need to be preserved in place. NOT for general vault note-taking (that's the hatchdoor skill).
---

# Clipping a Web Article to the Vault

Turn an article URL into a vault note holding the article's text and its editorial
images, stored locally rather than hotlinked. Builds on the [[hatchdoor]] skill for
the vault mechanics — read that skill for filing decisions, tagging, and the
image-import standard; this skill covers the article-specific parts: extraction,
image triage, and the upload sequence.

## Workflow

Upload the images **before** writing the note. Then the note is one write with the
final local paths in it, instead of a create followed by one `edit_note` per image
with a hash to chain.

1. **Extract the text, then check it is the whole article.** `tavily_extract` on
   the URL with `extract_depth: "advanced"`, `format: "markdown"`. This is the
   named extractor for article capture; use it for the prose. It sometimes returns
   a **partial** article with no error and no truncation marker: on a wired.com
   story (2026-09-10) it silently dropped roughly half the body, including two
   whole sections, keeping only the opening paragraph and the back half. Before
   transcribing anything, compare it against the full page text and use whichever
   is complete. A body that jumps from the intro straight to a later section, or
   that has fewer headings than the page, is the tell.
2. **Get the images from the page HTML.** Do not rely on Tavily's
   `include_images` — it returns nothing at all on some sites (wired.com,
   2026-09-09), so treat an empty image list as "wrong tool", not "no images".
   Fetch the page and walk it yourself:

   ```bash
   curl -sL --max-time 45 -A "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 Chrome/128.0 Safari/537.36" "$URL" -o /tmp/page.html
   ```

   ```python
   import re, html
   h = open('/tmp/page.html', encoding='utf-8', errors='replace').read()
   # <img> tags in document order; take the widest srcset candidate
   for tag in re.findall(r'<img\b[^>]*>', h):
       alt = re.search(r'\balt="([^"]*)"', tag)
       srcset = re.search(r'\bsrcset="([^"]*)"', tag)
       src = re.search(r'\bsrc="([^"]*)"', tag)
       ...
   # captions: pair each image with the caption element that FOLLOWS it
   caps = [(m.start(), re.sub(r'<[^>]+>', '', m.group(2)).strip())
           for m in re.finditer(r'<(figcaption|span)[^>]*[Cc]aption[^>]*>(.*?)</\1>', h, re.S)]
   # anchor on the LAST occurrence of the photo id, not the first
   pos = h.rfind(photo_id)
   caption = next((t for p, t in caps if p > pos), None)
   ```

   Pairing on document position is what recovers the per-photo credit lines
   ("Photograph: …", "Courtesy of Apple"), which differ image to image. Two traps:

   - **Anchor on the last occurrence of the photo id, not the first.** A CDN photo
     id appears dozens of times per page, and the first hit is in the JSON-LD or
     preload block at the top, hundreds of kilobytes before the body. Anchoring
     there hands every image the same caption. A correct pairing has a gap in the
     hundreds or low thousands of bytes; a gap in the hundreds of thousands means
     you anchored on metadata.
   - **An image with no caption steals the next one's.** "Nearest caption that
     follows" returns something for every image, including ones the page never
     captioned. Sanity-check the gaps, and cross-check against a rendered-markdown
     fetch of the page, which shows the caption lines where they actually sit.
3. **Triage.** Keep the editorial photos in the article body. Drop the author
   headshot, coupon/advertiser logos, "related content" thumbnails, nav icons and
   base64 data URIs. Two reliable tells on CDN-backed sites: a URL whose filename
   is literally `undefined` is a related-article card, and anything appearing after
   the last body heading is page furniture.
4. **Decide filing.** Use the router in the vault's Operating Rules note (loaded via
   the [[hatchdoor]] skill): fleet-ops → `homelab/`, everything else → `personal/`.
   For an explicitly temporary/throwaway note: file under `00-inbox/`, skip the
   `## Related` section, and skip cross-links entirely.
5. **Download and upload the images.** `curl` each keeper to a safe ASCII
   kebab-case filename, then:
   - `get_attachment_import_config` for the Vault. It reports the live methods,
     size limits and allowed extensions, and it is authoritative. **It is not a
     staging folder** — the sftp staging bridge was decommissioned 2026-08-29.
   - Upload with the **HTTP multipart endpoint** (the default, ≤10 MB): `POST
     /api/v1/vaults/<vault_id>/attachments`, multipart fields
     `target_relative_path` and `file`, `Authorization: Bearer <token>` where the
     token comes from `claude mcp get hatchdoor`. Fall back to `import_attachment`
     (base64, ≤5 MB) only where no shell is available.
   - Put them in a per-article subfolder beside the note:
     `<note-folder>/<article-slug>/<name>.jpg`.
6. **Write the note** in a single `create_note`. Transcribe the article: keep the
   heading structure, the byline, and each image in the position it held in the
   source, with its caption as an italic line underneath. Reference images by a
   path **relative to the note's folder** (`<article-slug>/<name>.jpg`), in plain
   Markdown `![alt](path)` form — never a wikilink embed, never URL-encoded. The
   note's `# H1` must match its filename, so the article's own headline becomes the
   first sub-heading rather than the H1.
7. **Verify the refs resolve.** `list_note_attachments` on the new note should
   return every image you uploaded. A ref that merely looks right in the source but
   resolves nowhere is absent from that list.
8. **Sync.** `sync_vault` to request an immediate turn, then `list_vaults` to
   confirm `git: ready` and `watcher: running`. There is no `get_git_sync_status`
   tool; it has never existed.

## Common mistakes

- Concluding an article has no images because `tavily_extract` returned none. Check
  the HTML before believing it.
- Importing decorative or promotional images alongside the real editorial photos.
- Leaving hotlinked external image URLs in the note as a "good enough" fallback. If
  an upload fails, diagnose it and say so; do not settle for hotlinks, and do not
  claim an image was imported when it was not.
- Creating the note first and then chaining an `edit_note` per image to swap the
  URLs. Each of those needs the hash returned by the *previous* write, and a stale
  hash rejects the edit. Upload first and the problem disappears.
- Attaching one blanket photo credit to every image. Credits vary per photo, and
  pairing each image with the caption that follows it in the HTML is what gets them
  right.

## Related
- [[hatchdoor]] — vault operating rules, filing router, image-import
  standard, git-sync discipline
