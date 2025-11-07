# Articles Directory

This directory is for manually added reference files that you want to keep in your project.

## Usage

1. Add your article files (`.txt`, `.md`, `.pdf`) to this directory
2. Run `/rgn.add-reference --process-local` to process all files
3. Or process a specific file: `/rgn.add-reference --process-local filename.pdf`

## Supported Formats

- **Text files** (`.txt`): Plain text articles
- **Markdown files** (`.md`): Markdown-formatted content
- **PDF files** (`.pdf`): PDF documents (text will be extracted)

## Processing

When you run the command with `--process-local`:
- Files remain in this directory (not copied elsewhere)
- Summaries are generated in `../summaries/`
- Mindmaps can be created with `--mindmap` flag
- Entries are added to `../References.md`

## Example

```bash
# Add your file
cp ~/Downloads/important-paper.pdf .memory/references/articles/

# Process it
/rgn.add-reference --process-local important-paper.pdf --tags "research,ai" --mindmap
```

## Notes

- Files are referenced from this location in References.md
- Original files are preserved and never modified
- You can organize files in subdirectories if needed
