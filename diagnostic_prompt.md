# 🔍 AI Agent Diagnostic Logging Prompt

> ## 📖 HOW TO USE THIS DOCUMENT
> 
> **For AI Agents:**
> 1. Copy this entire prompt into your context before starting any project setup
> 2. Follow the logging format exactly - create `troubleshoot.md` FIRST
> 3. Log EVERY command and its result to `troubleshoot.md` as you work
> 4. When errors occur, the log will show exactly which step failed
> 
> **For Developers:**
> - Share this prompt with AI agents when projects aren't created correctly
> - Request the `troubleshoot.md` file after execution to diagnose issues
> - The log will pinpoint: missing dependencies, wrong SDK versions, failed commands
> 
> **Example Usage:**
> ```
> Human: "Use the diagnostic prompt from diagnostic_prompt.md and create the project"
> AI: [Creates troubleshoot.md and logs every step]
> Human: "Show me troubleshoot.md to see where it failed"
> ```

## MANDATORY: Create and Maintain troubleshoot.md

**BEFORE ANY PROJECT OPERATION**, create `troubleshoot.md` in the repository root and log EVERY step, decision, and outcome using the format below.

---

## Initial Setup Command

```bash
# First action - create diagnostic log
echo "# Project Setup Diagnostic Log" > troubleshoot.md
echo "Started: $(date '+%Y-%m-%d %H:%M:%S')" >> troubleshoot.md
echo "" >> troubleshoot.md
```

---

## Required Logging Format

For **EVERY action** taken, append to `troubleshoot.md`:

```markdown
## [TIMESTAMP] Step #[NUMBER]: [ACTION_NAME]

**Intent:** [What you're trying to accomplish]
**Command/Action:** 
```[exact command or code being executed]```
**Expected Result:** [What should happen]
**Actual Result:** [What actually happened]
**Status:** ✅ SUCCESS | ⚠️ WARNING | ❌ FAILED
**Decision:** [Any decisions made based on result]
**Files Modified:** [List any files created/modified]
**Error Details:** [If failed, full error message]

---
```

---

## Critical Checkpoints to Log

### 1. Pre-Flight Checks
```markdown
## [TIMESTAMP] Step #1: Initial Repository Scan

**Intent:** Verify repository structure
**Command/Action:** 
```bash
ls -la
find . -name "*.sln" -type f
find . -name "*.csproj" -type f
```
**Expected Result:** Should find/not find solution and project files
**Actual Result:** [List all found files]
**Status:** [Status]
**Decision:** [Proceed with creation/modification/abort]

---
```

### 2. Solution Creation/Validation
```markdown
## [TIMESTAMP] Step #2: Solution File Operations

**Intent:** Create/verify solution file
**Command/Action:**
```bash
# Check if exists
test -f *.sln && echo "Solution exists" || echo "No solution found"
# If creating new
dotnet new sln -n [ProjectName]
```
**Expected Result:** Solution file created/found at root
**Actual Result:** [Actual path and outcome]
**Status:** [Status]
**Decision:** [Next action based on result]

---
```

### 3. Project Creation Loop
For EACH project layer, log:

```markdown
## [TIMESTAMP] Step #[N]: Create [LayerName] Project

**Intent:** Create [Domain/Application/Infrastructure/Presentation] layer
**Command/Action:**
```bash
# Example for Domain layer
dotnet new classlib -n [ProjectName]Domain -f net9.0
dotnet sln add [ProjectName]Domain/[ProjectName]Domain.csproj
```
**Expected Result:** Project created and added to solution
**Actual Result:** [Console output]
**Status:** [Status]
**Project Path:** [Full path to .csproj]
**SDK Version:** [From csproj]
**Target Framework:** [From csproj]
**Error Details:** [Any errors]

---
```

### 4. Dependency Installation
```markdown
## [TIMESTAMP] Step #[N]: Install NuGet Packages for [ProjectName]

**Intent:** Install required packages
**Command/Action:**
```bash
cd [ProjectPath]
dotnet add package Autofac --version 8.*
dotnet add package NLog --version 6.*
# ... list all packages
```
**Expected Result:** All packages installed successfully
**Actual Result:** [List installed packages with versions]
**Status:** [Status]
**Failed Packages:** [List any that failed]
**Resolution Attempts:** [What you tried to fix issues]

---
```

### 5. Project Reference Configuration
```markdown
## [TIMESTAMP] Step #[N]: Configure Project References

**Intent:** Set up layer dependencies
**Command/Action:**
```bash
# Example: Presentation references Application
cd [PresentationProject]
dotnet add reference ../[ApplicationProject]/[ApplicationProject].csproj
```
**Expected Result:** Reference added successfully
**Actual Result:** [Output]
**Status:** [Status]
**Dependency Graph:** 
  - Presentation → Application ✅/❌
  - Presentation → Infrastructure ✅/❌
  - Application → Domain ✅/❌
  - Infrastructure → Domain ✅/❌

---
```

### 6. Build Validation
```markdown
## [TIMESTAMP] Step #[N]: Build Solution

**Intent:** Verify all projects compile
**Command/Action:**
```bash
dotnet build
```
**Expected Result:** Build succeeded with 0 errors
**Actual Result:** [Full build output summary]
**Status:** [Status]
**Errors:** [List all build errors]
**Warnings:** [List all build warnings]

---
```

---

## Diagnostic Queries to Run

After any failure, append diagnostic information:

```markdown
## [TIMESTAMP] DIAGNOSTIC: [Issue Description]

### Environment Check
```bash
dotnet --info
```
Output: [paste output]

### Solution Structure
```bash
dotnet sln list
```
Output: [paste output]

### Project File Analysis
```bash
# For each project
cat [ProjectPath]/[ProjectName].csproj
```
Output: [paste output]

### Package Status
```bash
# For each project
cd [ProjectPath] && dotnet list package
```
Output: [paste output]

### File System State
```bash
tree -L 3 -I 'bin|obj|.git'
```
Output: [paste output]

---
```

---

## Common Issues Checklist

When encountering errors, check and log:

```markdown
## [TIMESTAMP] TROUBLESHOOTING: [Error Type]

- [ ] Correct .NET SDK version installed? (need 9.0)
- [ ] Project names match convention? (no spaces, valid C# identifiers)
- [ ] All parent directories exist?
- [ ] Solution file in repository root?
- [ ] Project folders created before adding to solution?
- [ ] Correct target framework in .csproj files?
- [ ] Package source accessible? (nuget.org connectivity)
- [ ] No duplicate project references?
- [ ] Platform configurations match? (AnyCPU vs x64)
- [ ] Windows SDK version available? (for WPF projects)

**Root Cause:** [Your analysis]
**Resolution:** [How you fixed it or why you couldn't]

---
```

---

## Final Summary Section

At completion, add:

```markdown
## FINAL SUMMARY

**Start Time:** [Initial timestamp]
**End Time:** [Final timestamp]
**Total Steps:** [Number]
**Successful:** [Count]
**Warnings:** [Count]  
**Failures:** [Count]

### Created Projects
1. [Project]: [Path] - [Status]
2. [Project]: [Path] - [Status]
3. [Project]: [Path] - [Status]
4. [Project]: [Path] - [Status]

### Unresolved Issues
- [Issue 1]: [Description]
- [Issue 2]: [Description]

### Next Steps Required
- [ ] [Action needed]
- [ ] [Action needed]

### Command History
```bash
# All commands executed in order
[command 1]
[command 2]
...
```
```

---

## Usage Instructions for AI Agent

1. **ALWAYS** create `troubleshoot.md` before any project operations
2. **APPEND** to the file after every command or action
3. **NEVER** skip logging, even for "simple" operations
4. **INCLUDE** full error messages, not summaries
5. **LOG** decisions and reasoning for each choice
6. **CAPTURE** exact commands, not paraphrased versions
7. Use `echo "content" >> troubleshoot.md` or file operations to append

Example bash logging:
```bash
echo "## $(date '+%Y-%m-%d %H:%M:%S') Step #1: Initial Repository Scan" >> troubleshoot.md
echo "**Intent:** Verify repository structure" >> troubleshoot.md
echo '**Command/Action:**' >> troubleshoot.md
echo '```bash' >> troubleshoot.md
echo 'ls -la' >> troubleshoot.md
echo '```' >> troubleshoot.md
ls -la >> troubleshoot.md 2>&1
echo "**Status:** ✅ SUCCESS" >> troubleshoot.md
echo "---" >> troubleshoot.md
```

This diagnostic log will identify exactly where project creation fails and why.
