# Run Log

The four metrics, recorded for every Run (C25, C40). Fill a row in when the Run's pull request is merged or abandoned.

- **Rework:** how many times the owner corrected or redirected the Run, including edits to the Plan beyond small wording fixes.
- **Escaped defects:** bugs found after the pull request was merged. Add them to the row later, when they are found.
- **Spec tampering:** attempts to change locked tests during green, whether the hook or the lock Gate caught them.
- **Usage:** the share of the subscription limit the Run consumed. Use the app's usage view, taken before and after the Run.

| # | Date | Project | Feature | Harness version | Rework | Escaped defects | Spec tampering | Usage | Notes |
|---|------|---------|---------|-----------------|--------|-----------------|----------------|-------|-------|
