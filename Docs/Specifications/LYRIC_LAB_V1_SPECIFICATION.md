# Lyric Lab — Version 1 Specification

## 1. Purpose

Lyric Lab V1 is a lightweight, Android-first lyric-writing application designed for fast, distraction-free writing.

The application behaves like a simple tabbed text editor, but with lyric-specific line numbering and session recovery.

The primary V1 goals are:

- Provide a clean lyric-writing surface.
- Number lyric lines by section rather than by document.
- Ignore blank lines for numbering.
- Restart numbering after blank lines.
- Support word wrapping without changing logical line counts.
- Autosave open working documents internally.
- Restore open tabs after the app is closed and reopened.
- Allow explicit saving to plain-text `.txt` files.
- Provide a Recent Documents screen when there are no open tabs.

V1 should remain deliberately simple. Features such as rhyme assistance, AI writing tools, cloud sync, settings, themes, album organization, and advanced formatting are outside V1 scope.

---

## 2. Platform

V1 is primarily intended for Android.

The implementation should use mobile-friendly interaction patterns and touch targets.

Desktop-specific behavior may be added later, but V1 interaction design should prioritize Android usability.

---

## 3. File Format

All user-facing saved lyric files must use plain-text `.txt` format.

Encoding:

- UTF-8

The saved `.txt` file must contain only the lyric text itself.

The following must **not** be written into the `.txt` file:

- Line numbers
- Tab titles
- Session metadata
- Autosave metadata
- Word-wrap information
- Internal document IDs

These values must be regenerated or restored from application metadata.

---

# 4. Lyric Editor

## 4.1 Editor Surface

The main writing area consists of:

1. A line-number gutter on the left.
2. A lyric text editor on the right.

The editor must support normal text entry, deletion, cursor movement, selection, copy, cut, paste, undo, and redo.

---

## 4.2 Logical Lines

A lyric line is defined by actual newline / end-of-line characters in the document.

A visually wrapped line is **not** a new logical line.

Example:

```text
1   This is a very long lyric line that reaches the edge of
    the editor and visually wraps onto another screen row
2   This is the next actual lyric line
```

The first lyric line is still logical line 1 because the user did not press Enter between the wrapped portions.

---

## 4.3 Word Wrap

Word wrap must always be enabled in V1.

Requirements:

- Long lines wrap automatically within the available editor width.
- The user must never need to horizontally scroll to read a long lyric line.
- Wrapped continuation rows receive no additional line number.
- Word wrap is presentation-only.
- Word wrap must never alter the saved text.

---

# 5. Section-Based Line Numbering

## 5.1 Core Rule

Line numbers apply only to consecutive non-empty lyric lines.

Blank lines are not numbered.

After one or more blank lines, numbering restarts at 1.

Example:

```text
1   First line of verse
2   Second line of verse
3   Third line of verse

1   First line of hook
2   Second line of hook

1   First line of another section
2   Second line of another section
```

---

## 5.2 Blank Lines

A blank logical line:

- Displays no line number.
- Ends the current numbering sequence.
- Causes the next non-empty logical line to restart at 1.

One blank line and multiple consecutive blank lines have the same numbering-reset effect.

---

## 5.3 Enter Key

Pressing Enter inserts a real newline character.

If the preceding line contains text, the next non-empty line continues the current numbering sequence.

Example:

```text
1   First line
2   Second line
3   Third line
```

If Enter is used to create a blank line, the numbering sequence ends.

Example:

```text
1   First line
2   Second line

1   New section
```

---

## 5.4 Dynamic Renumbering

Line numbers must update automatically whenever text is edited.

Examples:

### Deleting a blank line

Before:

```text
1   Line A
2   Line B

1   Line C
2   Line D
```

After deleting the blank line:

```text
1   Line A
2   Line B
3   Line C
4   Line D
```

### Turning a line into a blank line

Before:

```text
1   Line A
2   Line B
3   Line C
```

If `Line B` is deleted completely:

```text
1   Line A

1   Line C
```

The numbering system must always derive from the current document text rather than storing line numbers permanently.

---

# 6. Tabs

## 6.1 General Behavior

Each open document appears in its own tab.

The tab bar must:

- Display horizontally.
- Support horizontal scrolling.
- Highlight the active tab.
- Allow switching documents by tapping a tab.
- Preserve each tab's document state.
- Restore the tab order when the session is restored.

---

## 6.2 Tab Titles

Each tab has a display title stored separately from the file name.

The tab title and actual saved `.txt` filename do not need to match.

Example:

```text
Tab title: Midnight Drive
Saved filename: Song1.txt
```

A newly created document initially uses its generated untitled name as its tab title.

Example:

```text
Untitled 01
```

The user may later rename only the tab.

Renaming the tab must **not** rename the `.txt` file.

---

## 6.3 Renaming Tabs

Primary Android interaction:

1. Long-press a tab.
2. Show a context menu.
3. Include a `Rename` option.
4. Selecting Rename allows the tab title to be edited.
5. The existing title should be selected or made immediately editable.
6. The user confirms using the keyboard Done/Enter action or by completing the edit.

The renamed title is saved in application metadata.

---

## 6.4 Long Tab Names

Long titles must not consume excessive horizontal space.

Use truncation with ellipsis.

Example:

```text
Midnight Drive Through Johannesburg
```

may display as:

```text
Midnight Drive…
```

The full title remains stored internally.

---

## 6.5 Tab Reordering

Tabs must support drag-and-drop reordering.

On Android:

- The user long-presses and drags a tab.
- Nearby tabs shift to indicate the insertion position.
- Releasing the tab commits the new order.
- The new order is saved in session metadata.

The restored session must preserve the same tab order.

---

## 6.6 Closing Tabs

Every tab must contain a visible `×` close button.

The close target must be large enough for comfortable touch input.

Closing a tab is intentionally different from closing the entire app.

See Section 10 for close-tab rules.

---

# 7. New Documents

## 7.1 Creating a New Document

The File menu must include `New`.

Selecting New:

1. Creates a new document.
2. Opens it in a new tab.
3. Makes the tab active.
4. Assigns the next available generated name.
5. Creates an internal autosave working copy.

Naming pattern:

```text
Untitled 01
Untitled 02
Untitled 03
...
```

The numbering must continue safely without overwriting an existing document or internal working copy.

---

# 8. Autosave

## 8.1 Purpose

Autosave protects the user's active working session.

Autosave is **not** the same as explicitly saving a `.txt` file.

Open tabs must always have an internally autosaved working copy.

---

## 8.2 Autosave Behavior

Whenever document text changes:

- Schedule an autosave.
- Persist the latest working copy into Lyric Lab's internal application storage.

Autosave should appear continuous to the user.

Implementation should use a short debounce rather than performing unnecessary disk writes for every individual keystroke.

The debounce duration is an implementation detail, but it must be short enough that normal app closure or interruption does not risk meaningful recent work.

---

## 8.3 Internal Autosave Storage

Autosave working copies and application metadata should be stored in application-private storage.

On Windows-like conceptual paths this may be equivalent to:

```text
%LOCALAPPDATA%\Lyric Lab\
```

On Android, use the appropriate app-private storage location.

Do not store internal working copies in the public Documents folder unless platform requirements make that necessary.

---

# 9. Session Restore

## 9.1 Closing the Entire App

Closing the application must **not** prompt the user to save open tabs.

Reason:

- Every open tab already has an internal autosaved working copy.

When the app is reopened, restore:

- All previously open tabs
- Each tab's text
- Tab order
- Active tab
- Tab title
- Associated external file path, if any
- Unsaved changes in the working copy

The user should be able to close the app and return later without losing their active writing session.

---

## 9.2 Session Metadata

Persist enough metadata to restore the session reliably.

Suggested metadata per document:

```text
DocumentId
TabTitle
AutosaveWorkingCopyPath
ExternalFilePath (nullable)
HasExplicitFile
LastExplicitSaveState or equivalent comparison information
LastOpenedTimestamp
LastModifiedTimestamp
TabOrder
IsActive
```

Exact internal schema is implementation-specific.

---

# 10. Closing an Individual Tab

Closing an individual tab is treated as an intentional decision to remove that document from the active session.

---

## 10.1 Never Explicitly Saved

If the document has never been explicitly saved as a `.txt` file, show:

### Dialog Title

```text
Save changes?
```

### Message

```text
Do you want to save this tab before closing?
```

### Buttons

Visually on Android:

```text
Cancel    No    Yes
```

`Yes` is the primary affirmative action.

### Behavior

#### Yes

- Open the system Save As dialog.
- If the user completes the save successfully:
  - Save the current contents to the selected `.txt` file.
  - Remove the internal working copy for the closed tab.
  - Close the tab.
- If the user cancels the Save As dialog:
  - Do not close the tab.
  - Preserve everything unchanged.

#### No

- Permanently discard the tab's current working content.
- Delete its internal autosave copy.
- Close the tab.

#### Cancel

- Close the confirmation dialog.
- Return to the tab.
- Do not modify or discard anything.

---

## 10.2 Explicitly Saved Document With Changes

If the document has an existing explicit `.txt` file and the working copy has changed since the last explicit save, show:

```text
Save changes?
```

The actions should be:

- Save
- Don't Save
- Cancel

### Save

Write the latest working text to the existing file, delete the closing tab's internal working copy, and close the tab.

### Don't Save

Do not modify the external file.

Discard the internal working copy and close the tab.

### Cancel

Return to the tab unchanged.

---

## 10.3 Explicitly Saved Document Without Changes

If the document has no changes since the last explicit save:

- Close immediately.
- No confirmation dialog is required.
- Delete the now-unneeded internal working copy for that closed tab.

---

# 11. File Menu

V1 should include a minimal File menu with:

- New
- Open
- Save
- Save As

No larger settings system is required for V1.

---

# 12. Open

Selecting `Open`:

1. Opens the Android/system file picker.
2. Allows the user to choose a `.txt` file.
3. Reads the file as UTF-8 plain text.
4. Opens it in a new tab.
5. Adds it to the application's known/recent document metadata.
6. Creates an internal working copy for autosave/session protection.

If the exact same file is already open:

- Do not open a duplicate tab.
- Switch to the existing tab.

---

# 13. Save

`Save` applies when a document already has an external file path.

Selecting Save:

- Writes the latest working copy to the existing `.txt` file.
- Updates the application's record of the last explicitly saved state.
- Keeps the tab open.
- Continues internal autosave afterward.

If the document has never been explicitly saved:

- Save should behave like Save As.

---

# 14. Save As

Selecting `Save As`:

1. Opens the system file picker / save dialog.
2. Allows the user to choose:
   - File name
   - Destination folder
3. Saves the current contents as UTF-8 plain text.
4. Records the selected external file path in metadata.
5. Updates the last explicitly saved state.

Important:

After Save As, ongoing editing still autosaves to the internal working copy.

Lyric Lab must **not** continuously overwrite the external `.txt` file on every keystroke.

The external file changes only when the user performs an explicit Save action or confirms Save while closing the tab.

---

# 15. Recent Documents Screen

## 15.1 When It Appears

The Recent Documents screen appears only when there are no open tabs to restore.

Examples:

- The user closes all tabs intentionally.
- The app starts and there is no active session.

If any tab remains part of the active session, restore the tab session instead.

---

## 15.2 What Appears in the List

The Recent Documents list contains only documents that have been explicitly saved as real files.

Unsaved internal working tabs do not appear in Recent Documents.

Why:

- Unsaved tabs remain part of the active session.
- If the user closes such a tab and chooses No, its contents are discarded.
- If the user closes it and chooses Yes, it becomes an explicitly saved file.

---

## 15.3 List Behavior

The list should:

- Be vertically scrollable.
- Include all known explicitly saved documents.
- Sort by most recently opened first.
- Allow tapping an item to open it.
- Preserve the document's known tab title when appropriate.

Suggested metadata fields:

```text
DocumentId
DisplayTitle
LastKnownFileName
LastKnownFilePath
LastOpenedTimestamp
LastModifiedTimestamp
```

---

## 15.4 Missing Files

If a recent item points to a file that no longer exists, do not silently remove it.

Show a dialog.

Suggested title:

```text
File not found
```

Suggested message:

```text
Lyric Lab can't find this file at its last known location.
Do you want to remove it from the recent documents list?
```

Buttons:

- Remove
- Cancel

### Remove

- Remove only the metadata entry from Recent Documents.
- Do not attempt any file deletion.

### Cancel

- Keep the item in the recent list.
- Close the dialog.

---

# 16. Behavior When Last Tab Is Closed

Do **not** automatically create a new untitled document.

When the last tab is closed successfully:

- Display the Recent Documents screen.

If there are no recent documents, show the same screen in an empty state with a clear New Document action.

A new document is created only when the user explicitly chooses New.

---

# 17. Undo and Redo

The editor must support standard undo and redo behavior.

V1 does not require a custom version-history system.

Undo/redo history may remain session-based.

---

# 18. Application Metadata

Metadata should be stored separately from user `.txt` files.

The metadata system is responsible for information such as:

- Internal Document ID
- Tab title
- Tab order
- Active tab
- External file path
- Whether the document has ever been explicitly saved
- Last explicitly saved state or checksum
- Autosave working-copy path
- Last opened timestamp
- Last modified timestamp
- Recent document information

JSON is acceptable for V1 if it remains reliable and easy to maintain.

A lightweight local database is also acceptable if justified by the implementation.

Do not put metadata inside the lyric `.txt` file.

---

# 19. Dirty / Changed State

Lyric Lab must know whether the current working copy differs from the last explicitly saved version.

This state is required for close-tab behavior.

Implementation options include:

- Content hash comparison
- Stored last-saved snapshot
- Dirty flag maintained carefully

The implementation must remain correct after:

- App restart
- Session restore
- Save
- Save As
- Undo
- Redo
- External-file reopen

A hash or equivalent persisted comparison is preferred over relying solely on an in-memory dirty flag.

---

# 20. Recommended Top-Level Android Layout

The detailed visual design may be refined later, but V1 should provide the following structure:

```text
┌─────────────────────────────────┐
│ App Bar / File Menu             │
├─────────────────────────────────┤
│ Horizontally Scrollable Tabs    │
├──────┬──────────────────────────┤
│ Line │                          │
│ Nos. │     Lyric Editor         │
│      │                          │
│      │                          │
└──────┴──────────────────────────┘
```

When no tabs are open:

```text
┌─────────────────────────────────┐
│ App Bar / File Menu             │
├─────────────────────────────────┤
│                                 │
│       Recent Documents          │
│                                 │
│       [scrollable list]         │
│                                 │
│       New Document              │
│                                 │
└─────────────────────────────────┘
```

---

# 21. V1 Out of Scope

Do not implement these unless explicitly requested later:

- AI lyric generation
- Rhyme suggestions
- Syllable counting
- Rhyme highlighting
- Beat playback
- BPM tools
- Audio recording
- Cloud synchronization
- User accounts
- Collaboration
- Albums/projects
- Advanced formatting
- Rich text
- Fonts/settings UI
- Themes
- Export to PDF
- Export to Word
- Search/replace beyond standard editor behavior
- Custom keyboard
- Version history
- Trash/recycle-bin recovery for discarded tabs
- Recovery of a tab after the user explicitly chooses No / Don't Save

---

# 22. Critical Business Rules

Codex must preserve the following rules exactly:

1. A logical lyric line exists only because of a real newline/end-of-line marker.
2. Word wrap never creates a new numbered lyric line.
3. Blank lines have no number.
4. The first non-empty line after one or more blank lines starts again at 1.
5. Line numbers are generated dynamically and are never stored inside the `.txt` file.
6. Closing the entire app never prompts for Save.
7. Closing the app preserves all open tabs using internal autosave/session storage.
8. Closing an individual tab may prompt for Save depending on its explicit save state and changes.
9. Choosing No / Don't Save while closing a tab permanently discards that tab's internal working copy.
10. An unsaved tab that remains open is restored on next application launch.
11. Unsaved tabs do not appear in Recent Documents.
12. Recent Documents appears only when there are no open tabs to restore.
13. Save As creates or selects an external `.txt` file but ongoing typing continues to autosave internally.
14. External `.txt` files are updated only through explicit Save/Save As/close-and-save actions.
15. Tab titles are metadata and are independent of filenames.
16. Tab order is user-reorderable and must survive restart.
17. If a recent file is missing, ask the user before removing its metadata entry.
18. All external files use UTF-8 plain text.

---

# 23. Acceptance Criteria

## Editor

- [ ] User can type plain text lyrics.
- [ ] Long lyric lines word-wrap.
- [ ] Wrapped rows do not receive extra numbers.
- [ ] Blank lines display no number.
- [ ] Numbering restarts at 1 after blank lines.
- [ ] Removing blank lines correctly merges numbering sequences.
- [ ] Adding blank lines correctly splits numbering sequences.
- [ ] Undo and redo work.

## Tabs

- [ ] Multiple documents can remain open simultaneously.
- [ ] Tabs scroll horizontally.
- [ ] Active tab is visually identifiable.
- [ ] Tabs have visible close buttons.
- [ ] Tabs can be renamed independently of filenames.
- [ ] Tabs can be reordered using drag-and-drop.
- [ ] Tab order survives application restart.
- [ ] Long tab titles are truncated visually with ellipsis.

## Autosave and Restore

- [ ] Every open document has an internal autosave working copy.
- [ ] Text changes are autosaved automatically.
- [ ] Closing the entire app produces no save prompt.
- [ ] Reopening the app restores all previously open tabs.
- [ ] Restored tabs contain the latest autosaved content.
- [ ] Active tab and tab order are restored.

## File Operations

- [ ] New creates a new untitled tab.
- [ ] Open loads an existing UTF-8 `.txt` file.
- [ ] Opening an already-open file switches to its existing tab.
- [ ] Save writes to the current explicit file.
- [ ] Save on an unsaved document invokes Save As.
- [ ] Save As lets the user choose name and location.
- [ ] Continued editing after Save As does not continuously overwrite the external file.

## Tab Closing

- [ ] Closing a never-saved tab shows Yes / No / Cancel.
- [ ] Yes opens Save As.
- [ ] Cancelling Save As leaves the tab open.
- [ ] No discards the internal working copy permanently.
- [ ] Cancel leaves the tab unchanged.
- [ ] Closing a changed saved file offers Save / Don't Save / Cancel.
- [ ] Closing an unchanged saved file requires no confirmation.
- [ ] After the final tab closes, Recent Documents is shown.

## Recent Documents

- [ ] Screen appears only when there are no tabs to restore.
- [ ] Only explicitly saved files appear.
- [ ] All known recent files can be shown in a scrollable list.
- [ ] Most recently opened items appear first.
- [ ] Tapping a valid recent item opens it.
- [ ] Missing files trigger a File Not Found dialog.
- [ ] Missing entries are removed only after user confirmation.

---

# 24. Implementation Guidance for Codex

When implementing this specification:

1. Keep V1 intentionally small.
2. Do not add unrequested features.
3. Separate:
   - editor text
   - internal working-copy persistence
   - external user file persistence
   - session metadata
4. Treat autosave and explicit file save as different operations.
5. Write deterministic unit tests for the numbering algorithm.
6. Write tests for:
   - blank-line resets
   - multiple blank lines
   - wrapped lines
   - inserted/deleted newlines
   - close-tab decision flows
   - session restore
   - dirty-state detection
7. Ensure file operations do not block the UI thread.
8. Use atomic or otherwise safe metadata/autosave writes so a crash during persistence does not corrupt the entire session.
9. Preserve Android lifecycle state correctly.
10. Do not make product decisions that contradict the business rules above.

---

# 25. Suggested Initial Implementation Order

1. Application shell and tab container
2. Plain-text lyric editor
3. Section-based line-numbering engine
4. Word-wrap behavior
5. New-document creation
6. Internal autosave
7. Session metadata and restore
8. Tab rename
9. Tab reorder
10. Tab close flows
11. Open
12. Save
13. Save As
14. Recent Documents
15. Missing-file handling
16. Undo/redo verification
17. Automated tests
18. Android lifecycle and crash-recovery testing

---

# 26. Definition of V1 Complete

Lyric Lab V1 is complete when a user can:

1. Launch the app.
2. Create one or more lyric documents.
3. Type lyrics with word wrap.
4. See section-based line numbers that reset after blank lines.
5. Move between multiple tabs.
6. Rename and reorder tabs.
7. Close the app without manually saving.
8. Reopen the app and recover the full active session.
9. Explicitly save lyrics as UTF-8 `.txt` files.
10. Open existing `.txt` lyric files.
11. Close tabs using predictable Save / Don't Save / Cancel behavior.
12. Browse explicitly saved documents from Recent Documents when no tabs are open.

That is the full intended scope for Version 1.
