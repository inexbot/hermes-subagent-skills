# Git Conflict Markers Causing "File Not Found" in Static Site Generators

## The Pattern

Static site generators (VitePress, Hugo, Docusaurus, MkDocs, etc.) report errors like:

```
[vitepress] ENOENT: no such file or directory, stat '/path/to/<<<<<<< HEAD'
file: /path/to/扫描IP功能.md
```

The file exists. The directory exists. But the tool reports "no such file or directory" for a path that looks like it contains `<<<<<<< HEAD` or similar Git conflict markers.

## Root Cause

During a Git merge with conflicts, the conflict resolution was not completed. The file contains unresolved Git conflict markers:

```markdown
<<<<<<< HEAD
(their changes)
=======
(your changes)
>>>>>>> <branch-name>
```

When a static site generator scans the filesystem, it reads the raw file content and interprets the conflict markers as **part of a file path string**, then calls `stat()` on that path — which of course doesn't exist, because it's not a real file.

This commonly happens when:
- A merge conflict occurred in a `.md` file
- The conflict was visually resolved in an editor but the markers were not actually removed
- The file was saved and committed (or left uncommitted) in this state
- The file may show as "untracked" or "modified" in `git status`

## Diagnosis

```bash
# Check for conflict markers in all markdown files
grep -r "<<<<<<< HEAD\|=======$\|>>>>>>> " docs/ --include="*.md" -l

# Also check for incomplete conflict markers (only one side present)
grep -rn "<<<<<<<\|>>>>>>> " docs/ --include="*.md"
```

## Fix

1. Open each file containing conflict markers
2. Decide which version (or merged version) to keep
3. Remove ALL three types of conflict markers:
   - `<<<<<<< HEAD` (or `<<<<<<< <branch-name>`)
   - `=======`
   - `>>>>>>> <branch-name>` (or `>>>>>>>`)
4. Verify no markers remain: re-run the `grep` above
5. Rebuild

## Prevention

- Always resolve Git conflicts completely before building
- Use `git status` and `git diff` to confirm no conflict markers remain
- Consider a pre-build hook: `grep -r "<<<<<<<" docs/ && echo "Conflicts unresolved!" && exit 1`
