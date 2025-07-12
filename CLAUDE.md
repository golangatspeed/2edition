# Go Book Update Project - Second Edition

## Project Overview

**Objective**: Update an existing Go programming book from Go 1.19 to the latest version (Go 1.25), creating a comprehensive second edition that maintains the author's original style while incorporating new features, fixing errors, and improving content.

**Approach**: Collaborative, iterative process working chapter-by-chapter with careful review at each step.

## Project Context

### Original Book Details
- **Current Version**: Go 1.19 (now outdated)
- **Target Version**: Go 1.25
- **Format**: HTML version (for easy analysis and style extraction)
- **Status**: Needs comprehensive update for current Go features and best practices

### Author Constraints & Requirements
- **Time Availability**: Limited (works full-time)
- **Review Process**: Wants to review all changes before approval
- **Style Preservation**: Critical to maintain original writing voice and tone
- **Quality Goals**: Fix errors, update outdated content, add new Go 1.25 features

## Workflow Strategy

### Chosen Approach: Claude Code with Persistent Files
- **Tool**: Claude Code (research preview)
- **Rationale**: Maintains context between sessions via persistent file system
- **Structure**: Organized project directory with clear file hierarchy

### Project Structure
```
go-book-update/
├── CLAUDE.md              # This file - project brief and progress
├── original/
│   └── index.html        # Original book HTML version
├── style-guide.md        # Author's writing style analysis
├── go-changes.md         # Go 1.19 → 1.25 feature updates
├── chapters/
│   ├── ch01-revised.md   # Revised chapters
│   ├── ch02-revised.md
│   └── ...
├── review/
│   ├── changes-log.md    # Log of all changes made
│   └── review-notes.md   # Author feedback and decisions
└── progress.md           # Current status and next steps
```

## Key Discussion Points

### Technical Context
- **MCP Server Discussion**: Prior conversation about Morpheus MCP server and HTTPS configuration
- **Go Expertise**: Author clearly has deep Go knowledge (wrote comprehensive book)
- **Current Go Version**: Go 1.25 includes significant updates since 1.19

### Work Process
1. **Session-based**: Work in manageable chunks (1-2 chapters per session)
2. **Context Maintenance**: Use persistent files to track progress
3. **Style Analysis**: Analyze original writing to maintain consistency
4. **Review Cycles**: Author reviews each update before proceeding
5. **Incremental Progress**: Build up the second edition systematically

### Success Criteria
- [ ] Maintain author's original voice and style
- [ ] Update all Go 1.19 content to Go 1.25
- [ ] Fix any errors or outdated information
- [ ] Add coverage of new Go 1.25 features
- [ ] Ensure technical accuracy throughout
- [ ] Create a polished second edition ready for publication

## Next Steps

1. **Setup**: Author uploads original HTML to Claude Code project ✅
2. **Style Analysis**: Analyze writing patterns, tone, examples style ✅
3. **Go Changes Research**: Document major changes from Go 1.19 to 1.25 ✅
4. **Chapter Prioritization**: Decide which chapters to tackle first ✅
5. **Begin Revision**: Start systematic chapter-by-chapter updates

## Notes & Decisions

### Workflow Considerations
- **Context Preservation**: Critical requirement solved by Claude Code's persistent files
- **Literary Capabilities**: Confirmed Claude Code has same writing/editing abilities as web Claude
- **Version Control**: Natural git integration for tracking changes
- **File Management**: Ability to maintain organized project structure

### Author Preferences
- Prefers gradual, reviewable progress over bulk changes
- Values maintaining original writing style highly
- Wants comprehensive updates, not just patch fixes
- Comfortable with technical tools and command-line interfaces

### Code Examples Approach
- **Repository**: https://github.com/golangatspeed/GoFasterExamples
- **Pattern**: All code examples linked to Go Playground for interactive learning
- **Organization**: Chapter-based structure (chapter_1 through chapter_11) with numbered subdirectories
- **Naming**: Descriptive, hyphen-separated names with clear concept identification
- **Style**: Minimal, focused examples demonstrating specific concepts, pitfalls, and best practices
- **Requirement**: Maintain this approach for all revisions and new content in second edition

### Integration Strategy (Decided)
- **Hybrid Approach**: Integrate most new Go 1.20-1.25 features into existing 11 chapters
- **New Chapters**: Add Chapter 12 (Performance Optimization) and optionally Chapter 13 (Modern Web & API Development)
- **Feature Mapping**: Documented in go-changes.md with specific chapter assignments

## Progress Tracking

**IMPORTANT**: 
- **Session Start**: ALWAYS review progress.md file at the beginning of each new session to understand current status and pick up where we left off
- **Session End**: Update progress.md file after each chapter revision and major decision to maintain session continuity and track overall project status

---

**Status**: Chapters 6 & 7 complete - Two major revisions delivered
**Last Updated**: Current session  
**Next Action**: Author review of both chapters, then proceed with Chapter 8 or 10