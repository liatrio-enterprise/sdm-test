# Checkpoint Modes

Present these options to the user before starting implementation.

## Options

1. **Continuous Mode**: Pause after each sub-task (1.1, 1.2, 1.3)
   - Best for: Complex migrations, first-time GHA users
   - Maximum control and immediate feedback

2. **Task Mode** (Default): Pause after each parent task (1.0, 2.0, 3.0)
   - Best for: Standard migrations
   - Balance of control and momentum

3. **Batch Mode**: Pause after all tasks complete
   - Best for: Experienced users, straightforward migrations
   - Maximum momentum

**Default**: Task Mode if user doesn't specify.
