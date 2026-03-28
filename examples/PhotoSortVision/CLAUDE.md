# PhotoSortVision

This project uses the Test Pilot Loop framework.

@FLIGHT_DECK/AUTOPILOT.md

## Project-Specific

- **Platform:** macOS 14+ (Sonoma)
- **Tech Stack:** SwiftUI, Vision framework, Core Image
- **PRD:** PRD.md
- **Build:** `xcodebuild -scheme PhotoSortVision -destination 'platform=macOS'`
- **Run:** `open -a PhotoSortVision` or build and run from Xcode

### Conventions
- SwiftUI views in `Views/`
- Business logic in `Models/`
- No third-party packages
- All file operations use `FileManager` with undo support
