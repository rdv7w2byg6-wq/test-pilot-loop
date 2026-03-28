# Preflight Check — PhotoSortVision

Lives at `FLIGHT_DECK/PREFLIGHT_CHECK.md` in your project.

## Testing Depth

| Setting | Value |
|---------|-------|
| **Tiers** | Cold + Insider (skip Guided — no user manual exists) |
| **Personas** | Jordan (Everyday User) |
| **Dimensions** | All 6 |
| **Pass threshold** | 24/30 (80%) |
| **Mode** | Canonical — Cowork Opus sequential rotation |

## Dimension Weights

| Dimension | Weight | Notes |
|-----------|--------|-------|
| Usability & Clarity | 5 | Primary concern — Cold User must complete task |
| Functional Correctness | 5 | Files must actually move to correct folders |
| Navigation & Flow | 5 | Linear flow — import → dedupe → sort → done |
| Visual Design | 5 | Standard macOS look, nothing custom |
| Error Handling | 5 | What happens with no EXIF data, read-only folders, 0 photos? |
| Data Integrity | 5 | No photos lost, undo works, duplicates correctly identified |
