# Phase 1: Reduce Early Confusion - Implementation Spec

**Status:** Ready for Implementation
**Target Release:** Within 2 weeks
**Success Metric:** D1 retention improves from 22% to 25%+

---

## Overview

Phase 1 focuses on four high-impact, low-to-medium effort changes that reduce confusion in the first few minutes of gameplay.

---

## Task 1.1: Hide Multiplayer Tab for New Players

### Problem
The "Play | Multiplayer" toggle with crown icon is visible immediately, signaling "this is a competitive game" to players who came for a collection game.

### Solution
Hide the Multiplayer tab entirely for players under 25,000 lifetime taps.

### Implementation

```dart
// Location: Likely in main_screen.dart or home_screen.dart
// In the top navigation/header widget

Widget buildTopNavigation(Player player) {
  // NEW: Check if player is new
  final bool isNewPlayer = player.lifetimeTaps < 25000;

  if (isNewPlayer) {
    // Show simple header without toggle
    return Container(
      child: Center(
        child: Text(
          'One in a Trillion',
          style: TextStyle(/* your styling */),
        ),
      ),
    );
  } else {
    // Show existing Play/Multiplayer toggle
    return PlayMultiplayerToggle(/* existing implementation */);
  }
}
```

### Alternative: Locked State
If hiding completely feels too aggressive, show multiplayer as locked:

```dart
if (isNewPlayer) {
  return Row(
    children: [
      PlayButton(active: true),
      LockedMultiplayerButton(
        label: 'Multiplayer',
        unlockMessage: 'Unlocks at 25k taps',
        currentTaps: player.lifetimeTaps,
      ),
    ],
  );
}
```

### Edge Cases
- **Player logs in immediately:** Still base on lifetimeTaps, not login status
- **Player reinstalls:** lifetimeTaps should persist with account, so they see full UI
- **Player reaches 25k mid-session:** Show multiplayer immediately (celebration optional)

### Acceptance Criteria
- [ ] Players with < 25k taps do NOT see Multiplayer tab/toggle
- [ ] Players with >= 25k taps see normal UI
- [ ] Transition at 25k taps is smooth (no jarring UI change)

---

## Task 1.2: Add "Find 5 Eggs" First-Session Goal

### Problem
New players have no explicit goal. They see stats (taps, Egglopedia %) but don't know what they're trying to achieve.

### Solution
Show a clear first-session goal immediately after the nail tap tutorial.

### Implementation - Goal Overlay

```dart
// Location: Create new file goal_overlay.dart or add to tutorial system

class FirstSessionGoal {
  static const int TARGET_EGGS = 5;

  static void showIfNeeded(BuildContext context, Player player) {
    // Only show once, only for very new players
    if (player.lifetimeTaps < 500 && !player.hasSeenFirstGoal) {
      showDialog(
        context: context,
        barrierDismissible: false,
        builder: (context) => FirstGoalDialog(
          onDismiss: () {
            player.hasSeenFirstGoal = true;
            // Persist this flag
            savePlayerFlag('hasSeenFirstGoal', true);
            Navigator.pop(context);
          },
        ),
      );
    }
  }
}

class FirstGoalDialog extends StatelessWidget {
  final VoidCallback onDismiss;

  const FirstGoalDialog({required this.onDismiss});

  @override
  Widget build(BuildContext context) {
    return Dialog(
      backgroundColor: Colors.transparent,
      child: Container(
        padding: EdgeInsets.all(24),
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [Color(0xFF8B5CF6), Color(0xFFEC4899)],
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
          ),
          borderRadius: BorderRadius.circular(20),
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(
              '🎯 YOUR FIRST MISSION',
              style: TextStyle(
                color: Colors.white,
                fontSize: 18,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 16),
            Text(
              'Find 5 different eggs!',
              style: TextStyle(
                color: Colors.white,
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 16),
            // Progress circles
            Row(
              mainAxisAlignment: MainAxisAlignment.center,
              children: List.generate(5, (index) =>
                Container(
                  margin: EdgeInsets.symmetric(horizontal: 4),
                  width: 24,
                  height: 24,
                  decoration: BoxDecoration(
                    shape: BoxShape.circle,
                    border: Border.all(color: Colors.white, width: 2),
                  ),
                ),
              ),
            ),
            SizedBox(height: 24),
            ElevatedButton(
              onPressed: onDismiss,
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.white,
                foregroundColor: Color(0xFF8B5CF6),
                padding: EdgeInsets.symmetric(horizontal: 32, vertical: 12),
              ),
              child: Text(
                "Let's Go!",
                style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Implementation - Persistent Progress Indicator

```dart
// Location: Add to main gameplay screen, in the PROGRESS section

class SessionGoalProgress extends StatelessWidget {
  final int eggsFoundThisSession;
  final int targetEggs;
  final bool isNewPlayer;

  const SessionGoalProgress({
    required this.eggsFoundThisSession,
    required this.targetEggs,
    required this.isNewPlayer,
  });

  @override
  Widget build(BuildContext context) {
    // Only show for new players who haven't completed the goal
    if (!isNewPlayer || eggsFoundThisSession >= targetEggs) {
      return SizedBox.shrink();
    }

    return Container(
      padding: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Text(
            'Eggs: ',
            style: TextStyle(color: Colors.white70, fontSize: 14),
          ),
          ...List.generate(targetEggs, (index) {
            final bool found = index < eggsFoundThisSession;
            return Container(
              margin: EdgeInsets.symmetric(horizontal: 2),
              child: found
                ? Text('🥚', style: TextStyle(fontSize: 18))
                : Text('○', style: TextStyle(color: Colors.white38, fontSize: 18)),
            );
          }),
          Text(
            ' (${eggsFoundThisSession}/${targetEggs})',
            style: TextStyle(color: Colors.white70, fontSize: 14),
          ),
        ],
      ),
    );
  }
}
```

### Implementation - Goal Completion Celebration

```dart
// When player finds their 5th unique egg

void onEggCollected(Player player, Egg egg) {
  // Existing egg collection logic...

  // NEW: Check for first-session goal completion
  final uniqueEggsThisSession = player.getUniqueEggsThisSession();

  if (uniqueEggsThisSession == 5 && !player.hasCompletedFirstGoal) {
    player.hasCompletedFirstGoal = true;
    savePlayerFlag('hasCompletedFirstGoal', true);

    // Show celebration
    showGoalCompletionCelebration(context, player);
  }
}

void showGoalCompletionCelebration(BuildContext context, Player player) {
  showDialog(
    context: context,
    builder: (context) => Dialog(
      child: Container(
        padding: EdgeInsets.all(24),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text('🎉 MISSION COMPLETE!', style: TextStyle(fontSize: 24)),
            SizedBox(height: 16),
            Text('You found 5 eggs!'),
            SizedBox(height: 8),
            Text(
              'Keep exploring to find 100+ more rare eggs!',
              textAlign: TextAlign.center,
            ),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => Navigator.pop(context),
              child: Text('Continue'),
            ),
          ],
        ),
      ),
    ),
  );
}
```

### Copy/Messaging

**Goal Overlay:**
- Title: "🎯 YOUR FIRST MISSION"
- Goal: "Find 5 different eggs!"
- Button: "Let's Go!"

**Progress Display:**
- Format: "Eggs: 🥚🥚○○○ (2/5)"

**Completion:**
- Title: "🎉 MISSION COMPLETE!"
- Message: "You found 5 eggs!"
- Follow-up: "Keep exploring to find 100+ more rare eggs!"

### State Management

New flags needed:
```dart
class Player {
  // Existing fields...

  // NEW: First-session goal tracking
  bool hasSeenFirstGoal = false;
  bool hasCompletedFirstGoal = false;
  Set<String> uniqueEggsThisSession = {};

  int getUniqueEggsThisSession() => uniqueEggsThisSession.length;

  void trackEggFound(Egg egg) {
    uniqueEggsThisSession.add(egg.id);
  }

  void resetSession() {
    uniqueEggsThisSession.clear();
  }
}
```

### Acceptance Criteria
- [ ] Goal overlay shows after nail tap tutorial for players < 500 taps
- [ ] Goal overlay only shows once (persisted flag)
- [ ] Progress indicator shows on main screen for players working on goal
- [ ] Celebration shows when 5 unique eggs found
- [ ] Progress indicator hides after goal completed
- [ ] Unique eggs tracked per session (resets on app close/reopen)

---

## Task 1.3: Hide Daily Best Speed for New Players

### Problem
Two speed numbers (current + daily best) are confusing without context.

### Solution
Show only current speed for players under 10,000 lifetime taps.

### Implementation

```dart
// Location: Speed display widget

class SpeedDisplay extends StatelessWidget {
  final int currentSpeed;
  final int dailyBestSpeed;
  final int lifetimeTaps;

  @override
  Widget build(BuildContext context) {
    final bool isNewPlayer = lifetimeTaps < 10000;

    return Container(
      padding: EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      decoration: BoxDecoration(
        color: Colors.white.withOpacity(0.2),
        borderRadius: BorderRadius.circular(12),
      ),
      child: Column(
        children: [
          Text(
            'SPEED',
            style: TextStyle(
              color: Colors.white,
              fontSize: 12,
              fontWeight: FontWeight.bold,
            ),
          ),
          Text(
            '$currentSpeed',
            style: TextStyle(
              color: Colors.white,
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),
          // NEW: Only show daily best for experienced players
          if (!isNewPlayer) ...[
            Row(
              mainAxisSize: MainAxisSize.min,
              children: [
                Text('🔥', style: TextStyle(fontSize: 12)),
                SizedBox(width: 4),
                Text(
                  '$dailyBestSpeed',
                  style: TextStyle(color: Colors.white70, fontSize: 14),
                ),
              ],
            ),
          ],
        ],
      ),
    );
  }
}
```

### First-Time Tooltip (Optional Enhancement)

```dart
// Show once when player first sees speed
if (lifetimeTaps > 100 && !player.hasSeenSpeedTooltip) {
  showTooltip(
    message: 'Your tap speed! Collect items to go faster.',
    onDismiss: () {
      player.hasSeenSpeedTooltip = true;
      savePlayerFlag('hasSeenSpeedTooltip', true);
    },
  );
}
```

### Acceptance Criteria
- [ ] Players with < 10k taps see only current speed (no flame icon/daily best)
- [ ] Players with >= 10k taps see both speeds (existing behavior)
- [ ] Transition is clean (daily best appears naturally after 10k)

---

## Task 1.4: Show Absolute Egg Count Instead of Percentage

### Problem
"Egglopedia 0.41%" feels like failure. Small percentages are demotivating.

### Solution
For new players, show "Eggs Found: X" instead of percentage.

### Implementation

```dart
// Location: Progress section on main screen

class EgglopediaProgress extends StatelessWidget {
  final int uniqueEggsFound;
  final double egglopediaPercent;
  final int lifetimeTaps;

  @override
  Widget build(BuildContext context) {
    final bool isNewPlayer = lifetimeTaps < 50000;

    return Row(
      children: [
        Text('🌐', style: TextStyle(fontSize: 16)),
        SizedBox(width: 8),
        if (isNewPlayer) ...[
          // NEW: Show absolute count for new players
          Text(
            'Eggs Found: $uniqueEggsFound',
            style: TextStyle(
              color: Colors.white,
              fontSize: 24,
              fontWeight: FontWeight.bold,
            ),
          ),
        ] else ...[
          // Existing: Show percentage for veterans
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                'Egglopedia',
                style: TextStyle(color: Colors.white70, fontSize: 12),
              ),
              Text(
                '${egglopediaPercent.toStringAsFixed(2)}%',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                ),
              ),
            ],
          ),
        ],
      ],
    );
  }
}
```

### Alternative: Show Both

```dart
if (isNewPlayer) {
  return Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text(
        '$uniqueEggsFound eggs found',
        style: TextStyle(color: Colors.white, fontSize: 20, fontWeight: FontWeight.bold),
      ),
      Text(
        '100+ more to discover!',
        style: TextStyle(color: Colors.white70, fontSize: 12),
      ),
    ],
  );
}
```

### Acceptance Criteria
- [ ] Players with < 50k taps see "Eggs Found: X" (absolute number)
- [ ] Players with >= 50k taps see "Egglopedia X%" (existing behavior)
- [ ] The number feels like an achievement, not a failure

---

## Testing Checklist

### Before Release
- [ ] Test with fresh account (0 taps)
- [ ] Test at each threshold (500, 10k, 25k, 50k taps)
- [ ] Test flag persistence (close app, reopen)
- [ ] Test goal progress tracking (unique eggs per session)
- [ ] Test goal completion celebration
- [ ] Verify no crashes or UI glitches

### After Release
- [ ] Monitor crash reports
- [ ] Document baseline D1 retention before release
- [ ] Wait 14 days
- [ ] Measure D1 retention
- [ ] Compare to baseline

---

## Rollback Plan

If D1 retention drops below 20% after Phase 1:

1. Immediately revert all Phase 1 changes
2. Investigate which specific change may have caused issues
3. Consider releasing changes individually with longer measurement periods

---

## Files to Modify (Estimated)

| File | Changes |
|------|---------|
| main_screen.dart (or equivalent) | Add new player checks, goal progress display |
| navigation/header widget | Hide multiplayer for new players |
| speed_display.dart | Conditional daily best display |
| progress_section.dart | Conditional Egglopedia display |
| player.dart / player_model.dart | Add new flags and session tracking |
| NEW: first_goal_overlay.dart | Goal dialog and celebration |

---

## Timeline

| Day | Task |
|-----|------|
| 1-2 | Implement Task 1.1 (Hide Multiplayer) |
| 2-3 | Implement Task 1.3 & 1.4 (Speed & Egglopedia display) |
| 3-5 | Implement Task 1.2 (First-session goal - most complex) |
| 6-7 | Testing and polish |
| 8 | Release to production |
| 8-22 | Measurement period (14 days) |

---

## Success Criteria Summary

| Metric | Current | Target |
|--------|---------|--------|
| D1 Retention | 22% | 25%+ |
| Player confusion (qualitative) | High | Reduced |
| First-session goal completion | N/A | >50% of new players |
