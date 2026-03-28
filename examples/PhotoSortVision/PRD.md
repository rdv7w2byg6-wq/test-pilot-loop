# PhotoSortVision — Product Requirements

## Overview

A macOS menu bar app that imports photos from a selected folder, detects duplicates using perceptual hashing, and sorts originals into date-based subfolders.

## Target User

Casual Mac user with 500–5,000 unsorted photos in a single folder. Not technical. Expects drag-and-drop simplicity.

## Core Features

### F1: Photo Import
- User selects a source folder via standard macOS open panel
- App scans for `.jpg`, `.png`, `.heic` files recursively
- Progress indicator shows scan status
- Display total count when complete

### F2: Duplicate Detection
- Perceptual hash (pHash) comparison across all imported photos
- Group duplicates with a similarity threshold of 95%
- Display duplicate groups in a grid — user picks which to keep
- "Keep Best" auto-selects the highest resolution in each group

### F3: Date Sort
- Read EXIF `DateTimeOriginal` from each photo
- Sort into `YYYY/MM-MonthName/` folder structure
- Photos with no EXIF date go to `Unsorted/`
- Preview the proposed folder structure before executing
- "Sort Now" button executes the move

### F4: Undo
- All moves are reversible for 30 minutes after sorting
- "Undo Last Sort" button in the toolbar
- After 30 minutes, undo history is cleared with a notification

## Non-Functional Requirements

- macOS 14+ (Sonoma)
- Native SwiftUI, no third-party dependencies
- All processing on-device (no cloud)
- Handles 5,000 photos without UI freeze (background processing with progress)

## Out of Scope (v1)

- Video files
- iCloud Photos integration
- Batch rename
- Face detection grouping

## Success Criteria

A Cold User (no prior knowledge) can:
1. Open the app
2. Select a folder of photos
3. See and resolve duplicates
4. Sort photos by date
5. Undo if they made a mistake

...all without reading documentation or asking for help.
