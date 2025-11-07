---
description: Add and manage project references (articles, files, URLs, videos)
---

You are helping the user add a reference to their project. References can be articles, local files, URLs, or YouTube videos. Follow these steps carefully:

## Reference Processing Workflow

### 1. Parse Input and Options

Extract reference sources (single or multiple) and options from the user's command:
- **Reference sources**: Can be one or more of:
  - URLs (articles, web pages)
  - File paths (local files to copy)
  - YouTube video URLs
  - Multiple sources separated by spaces or commas
- `--mindmap` flag: whether to generate mindmaps for all references
- `--tags`: user-provided tags (applied to all references)
- `--process-local`: process files from `.memory/references/articles/` directory

**Multiple References:**

The command accepts multiple references in several formats:
```
/rgn.add-reference <url1> <url2> <file.pdf>
/rgn.add-reference url1, url2, file.pdf
/rgn.add-reference --tags "ai,ml" url1 url2 url3
```

When multiple references are provided:
- Process each reference sequentially
- Apply global flags (--mindmap, --tags) to all references
- Track progress and confirm each reference as processed
- Continue processing even if one reference fails (log errors)
- Provide summary at the end with success/failure counts

**Special case: `--process-local` option**

When this flag is used:
- Scan `.memory/references/articles/` for text, markdown, or PDF files
- Process each file found (or specific files if names provided)
- Skip the download/conversion step (files are already local)
- Create summaries and mindmaps as usual
- Reference files from their current location in `articles/` directory
- Can process all files: `/rgn.add-reference --process-local`
- Or specific files: `/rgn.add-reference --process-local file1.pdf file2.md`

### 2. Create Directory Structure

Ensure these directories exist:
```
.memory/references/
├── articles/         # Manually added files (text, md, pdf)
├── contents/         # Downloaded and converted content
├── summaries/        # AI-generated summaries
└── mindmaps/         # Conceptual mindmaps
```

Create them if they don't exist using the Bash tool.

### 3. Download and Process the Reference(s)

**Note:** If multiple references are provided, repeat steps 3-7 for each reference. Use the TodoWrite tool to track progress when processing multiple references.

**For locally added files (--process-local option):**
- Check if file exists in `.memory/references/articles/`
- Supported formats: `.txt`, `.md`, `.pdf`
- Read the file content using the Read tool
- For PDFs, extract text content
- For text/markdown, read directly
- Extract metadata:
  - Filename (use as title if no title found in content)
  - File modification date
  - Detect author from document if possible (look for "Author:", "By:", etc.)
  - File format
- **Important:** Do NOT copy to `contents/` directory
- Reference the file from its current location: `.memory/references/articles/<filename>`
- Continue to step 5 (Generate Summary) with the extracted content

**For URLs (articles, web pages):**
- Use WebFetch to retrieve the content
- Extract the main text content
- Identify the title, author (if available), and publication date

**For YouTube videos:**
- Extract video ID from URL (e.g., `https://www.youtube.com/watch?v=VIDEO_ID`)
- Attempt to download transcript using one of these methods (try in order):

  **Method 1: Using yt-dlp (preferred)**
  ```bash
  yt-dlp --write-auto-sub --skip-download --sub-format vtt --convert-subs srt --output ".memory/references/contents/%(title)s.%(ext)s" "VIDEO_URL"
  ```
  - This downloads auto-generated or manual subtitles
  - If successful, read the .srt or .vtt file and convert to markdown

  **Method 2: Using youtube-transcript-api (Python)**
  ```bash
  pip install youtube-transcript-api
  python -c "from youtube_transcript_api import YouTubeTranscriptApi; import sys; print('\n'.join([x['text'] for x in YouTubeTranscriptApi.get_transcript('VIDEO_ID')]))"
  ```
  - If available, use this to fetch transcript programmatically

  **Method 3: Manual extraction**
  - If tools aren't available, use WebFetch to access the YouTube page
  - Extract video metadata: title, channel, description, upload date
  - Include description and available metadata as content
  - Note in the content that transcript was not available

- Fetch video metadata (title, author/channel, description, duration)
- Create markdown content with:
  - Video title and link
  - Channel/author name
  - Upload date
  - Video description
  - Transcript (if successfully downloaded)
  - Timestamps from transcript (preserve if available)

**For local files:**
- Read the file using the Read tool
- Extract metadata (filename, format)

**For PDFs:**
- Read the PDF and extract text content
- Preserve structure and headings

### 4. Convert to Markdown Format

Save the full content as a markdown file:
- Filename: `.memory/references/contents/<sanitized-title>.md`
- Include metadata header with:
  - Original URL/source
  - Author
  - Date accessed
  - Original format
- Convert content to clean markdown
- Preserve headings, lists, code blocks, and formatting

**For YouTube video content, use this structure:**
```markdown
---
title: [Video Title]
author: [Channel Name]
source: [YouTube URL]
date_accessed: [YYYY-MM-DD]
upload_date: [YYYY-MM-DD]
duration: [HH:MM:SS]
type: youtube_video
transcript_available: [yes/no]
---

# [Video Title]

**Channel:** [Channel Name]
**URL:** [YouTube URL]
**Duration:** [HH:MM:SS]
**Uploaded:** [Date]

## Description

[Video description from YouTube]

## Transcript

[Transcript content with timestamps if available]

[HH:MM:SS] Transcript text here...
[HH:MM:SS] More transcript text...

[If no transcript: Note that transcript was not available]
```

### 5. Generate Summary

Create a comprehensive summary and save to `.memory/references/summaries/<sanitized-title>.md`:
- **Overview**: 2-3 paragraphs explaining the key points
- **Key Concepts**: Bulleted list of main ideas
- **Actionable Insights**: What can be applied from this reference
- **Tags**: Auto-generated tags based on content (merge with user-provided tags)
- Include metadata: author, source, date

### 6. Create Mindmap (Optional)

If `--mindmap` flag is present, generate a mindmap:
- Save to `.memory/references/mindmaps/<sanitized-title>.txt`
- **Format:** Plain-text, tab-separated hierarchical structure
- **Required:** Must include one node with the original article address/URL

**Mindmap Structure:**
```
[Article Title]	[Original URL or file path]
	Main Topic 1
		Subtopic 1.1
			Detail 1.1.1
			Detail 1.1.2
		Subtopic 1.2
			Detail 1.2.1
	Main Topic 2
		Subtopic 2.1
		Subtopic 2.2
			Detail 2.2.1
	Main Topic 3
		Subtopic 3.1
```

**Rules:**
- First line must be: `[Title]\t[Source URL/Path]`
- Each level of indentation uses ONE tab character
- Root level (article) = no tabs
- First level topics = 1 tab
- Second level subtopics = 2 tabs
- Third level details = 3 tabs (and so on)
- No bullet points or markdown formatting
- Keep nodes concise (5-10 words max per node)
- Focus on conceptual relationships and key ideas
- Organize hierarchically from general to specific

### 7. Classify and Add to References.md

Determine the appropriate category/section based on content:
- Common categories: Research, Documentation, Tutorials, Tools, Theory, Implementation, Case Studies
- Create new categories if needed

Update `.memory/references/References.md` (create if doesn't exist):

**File Structure:**
```markdown
# Project References

## [Category Name]

### [Reference Title]

**TL;DR**

[2-3 paragraph summary highlighting the most important aspects and why this reference matters to the project]

**Metadata:**
- **Author:** [Author Name]
- **Source:** [Original URL or file path]
- **Date Added:** [YYYY-MM-DD]
- **Tags:** `tag1`, `tag2`, `tag3`

**Resources:**
- [Full Content](.memory/references/contents/<filename>.md) *(or for local files: .memory/references/articles/<filename>)*
- [Summary](.memory/references/summaries/<filename>.md)
- [Mindmap](.memory/references/mindmaps/<filename>.txt) *(if generated)*

---
```

### 8. Sanitization Rules

For filenames (sanitized-title):
- Convert to lowercase
- Replace spaces with hyphens
- Remove special characters except hyphens and underscores
- Limit to 50 characters
- Ensure uniqueness (append number if exists)

### 9. Confirmation

**For single reference:**
After processing, confirm to the user:
- Reference title and source
- Category assigned
- Files created (with relative paths)
- Tags applied
- Whether mindmap was generated

**For multiple references:**
After processing all references, provide a summary:
- Total number of references processed
- Success count and failure count
- List of successfully added references (titles)
- List of failed references with brief error reasons
- Common files created: summaries, mindmaps (if --mindmap used)
- Overall tags applied

Example confirmation format:
```
Processed 5 references:
✓ Successfully added: 4
✗ Failed: 1

Successfully added:
- "Understanding RAG Systems" (Research)
- "Introduction to Vector Databases" (Documentation)
- "LangChain Tutorial" (Tutorials)
- "Embedding Models Comparison" (Research)

Failed:
- https://broken-link.com/article - URL inaccessible

Files created:
- 4 summaries in .memory/references/summaries/
- 4 mindmaps in .memory/references/mindmaps/
- All entries added to .memory/references/References.md

Tags applied: ai, ml, rag
```

## Example Usage

```
# Single reference
User: /rgn.add-reference https://arxiv.org/abs/2301.xxxxx --mindmap
User: /rgn.add-reference https://www.youtube.com/watch?v=xxxxx
User: /rgn.add-reference ./papers/important-paper.pdf --tags "machine-learning,rag"

# Multiple references (space-separated)
User: /rgn.add-reference https://arxiv.org/abs/2301.xxxxx https://arxiv.org/abs/2302.yyyyy --mindmap
User: /rgn.add-reference url1 url2 url3 --tags "ai,ml"
User: /rgn.add-reference https://example.com/article1 ./paper.pdf https://youtube.com/watch?v=abc

# Multiple references (comma-separated)
User: /rgn.add-reference url1, url2, url3 --mindmap --tags "research"

# Mixed types
User: /rgn.add-reference https://blog.com/post ./local-file.pdf https://youtube.com/watch?v=xyz --tags "tutorial"

# Process locally managed files (from articles/ directory)
User: /rgn.add-reference --process-local
User: /rgn.add-reference --process-local article1.pdf article2.md --tags "research,ai"
User: /rgn.add-reference --process-local --mindmap

# Process all local files with options
User: /rgn.add-reference --process-local --mindmap --tags "research,papers"

# Note: With --process-local, files should be manually placed in:
# .memory/references/articles/
```

## Error Handling

**General errors:**
- If URL is inaccessible, log the error and continue with next reference
- If file doesn't exist, log the error and continue with next reference
- If content is too large, process in chunks
- If parsing fails, save raw content and note the issue

**Batch processing:**
- Continue processing remaining references even if one fails
- Track all errors and report at the end
- Do not stop the entire batch due to a single failure
- Provide detailed error summary with failed reference details

**YouTube-specific errors:**
- If transcript download fails, fall back to the next method
- If no transcript is available at all:
  - Save video metadata and description as content
  - Note in the markdown: "Transcript not available for this video"
  - Still create summary based on title and description
- If yt-dlp or youtube-transcript-api not installed:
  - Inform user they can install these tools for better transcript support
  - Use Method 3 (manual extraction) as fallback
  - Installation commands:
    - `pip install yt-dlp youtube-transcript-api`
    - Or system package manager for yt-dlp

**Local file processing errors:**
- If `.memory/references/articles/` directory is empty:
  - Inform user to add files to the directory first
  - Suggest: "Add your .txt, .md, or .pdf files to .memory/references/articles/ and run again"
- If specific file not found in articles/:
  - List available files in the directory
  - Ask user to confirm the correct filename
- If file format is unsupported:
  - List supported formats: .txt, .md, .pdf
  - Suggest converting the file to a supported format
- If PDF text extraction fails:
  - Note the issue in the summary
  - Still create entry with available metadata

## Important Notes

- Always use relative paths from project root in References.md
- Preserve original source attribution
- Generate meaningful tags based on content analysis
- Keep TL;DR concise but informative (2-3 paragraphs)
- Maintain consistent formatting across all references
- If References.md exists, preserve existing entries and formatting

**Local file processing (`--process-local`):**
- Files in `.memory/references/articles/` are never modified or moved
- These files are referenced in-place (not copied to `contents/`)
- Summaries and mindmaps are still generated normally
- Use this for files you want to keep organized in your project
- Subdirectories within `articles/` are supported
- The command can process all files at once or a specific file

**Batch processing multiple references:**
- Use the TodoWrite tool to create and track tasks for each reference
- Process references sequentially, updating todo status as you go
- Mark each reference as completed after successful processing
- Continue with remaining references even if one fails
- Provide a comprehensive summary at the end with success/failure counts
- Apply global flags (--mindmap, --tags) to all references in the batch
