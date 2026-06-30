---
name: github-skill-import
description: "Clone a GitHub repo containing Hermes skills (or skill documentation), read specified files, extract the SKILL.md schema/frontmatter format, and create new skills following that schema. End-to-end workflow for importing external skill patterns."
version: 1.0.0
metadata:
  hermes:
    tags: [github, skills, import, schema, workflow]
    category: dev
---

# GitHub Skill Import — Clone, Read, Extract Schema, Create Skills

## When to Use
Use this skill when the user provides:
- A GitHub repo URL containing skills, skill documentation, or skill templates
- A request to "clone these skills into memory" or "read files and create skills from them"
- A specification or guide that defines a schema format for skills
- A multi-part request that chains: clone → read specific files → extract schema → create skills → verify

## Procedure

### Step 1: Clone the Repository
```bash
git clone <repo-url>
```
If the repo was already cloned in this session, skip this step and use the existing directory.

### Step 2: Identify and Read Specified Files
Use `search_files(target="files", path="<repo>")` to discover files. For the requested files:
- Use `read_file()` to load each specified document
- If filenames are approximate, use `search_files(pattern="<keyword>", target="files", path="<repo>")` to locate them

### Step 3: Extract the Schema
From the documentation, identify:
- **Frontmatter fields** — name, description, version, metadata (tags, category)
- **Body sections** — When to Use (trigger conditions), Procedure (numbered steps), Pitfalls, Verification
- **Directory structure** — `~/.hermes/skills/<category>/<skill-name>/SKILL.md`

Output the schema cleanly for reference before creating skills.

### Step 4: Create Target Directories
```bash
mkdir -p ~/.hermes/skills/<category>/<skill-name>
```
Always ensure the parent directory exists before writing SKILL.md.

### Step 5: Write Skills Per Schema
Use `write_file()` to create each SKILL.md with:
- Valid YAML frontmatter (`---` delimiters)
- Full Procedure with numbered steps
- Explicit Pitfalls based on any errors encountered
- Verification section to confirm success

Use proper path: `C:\Users\<user>\AppData\Local\hermes\skills\<category>\<skill-name>\SKILL.md`

### Step 6: Verify Placement
```bash
ls -la ~/.hermes/skills/<category>/<skill-name>/SKILL.md
```
Confirm the files exist at the correct path with expected sizes.

## Pitfalls
- **MSYS/Git-bash path resolution** — `write_file` with a path like `/c/Users/...` resolves to `C:\c\Users\...` instead of `C:\Users\...`. Always verify after writing. If the file landed at the wrong path, recover with:
  ```bash
  cp /c/c/Users/soren/AppData/Local/hermes/skills/<cat>/<skill>/SKILL.md "/c/Users/soren/AppData/Local/hermes/skills/<cat>/<skill>/SKILL.md"
  rm -rf /c/c/Users/soren/AppData/Local/hermes/skills/<cat>
  ```
  To avoid the issue entirely: pass literal `C:\Users\...` strings to `write_file`, or write files via terminal with `cat > path <<'EOF' ... EOF`.
- **Cloning into existing directory** — if the repo directory already exists, `git clone` fails. Remove it first with `rm -rf <dir>` or skip the clone step.
- **Missing schema sections** — if the source docs don't clearly define a schema, infer from examples and note the gap in Pitfalls.
- **Multi-part request ordering** — when the user chains clone → read → schema → create → capture, do not skip intermediate outputs. Each step is a dependency for the next.

## Verification
1. `ls <repo>` shows the cloned repository.
2. `read_file` returns the requested documents with content.
3. Schema template is correct and complete.
4. `ls ~/.hermes/skills/<category>/<skill-name>/SKILL.md` confirms each skill was saved.
5. Re-read one skill file to validate frontmatter and body structure.
6. Confirm no files landed at a `C:\c\...` wrong-path artifact — `find ~/.hermes/skills -path "*\\c\\*" 2>/dev/null` should return empty.
