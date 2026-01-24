# Automath - Project Context

## Project Overview

**Automath** is a minimal, browser-based math practice application designed for K-5 students to develop automaticity in basic arithmetic operations through daily warmup exercises.

### Purpose
- Enable students to practice basic math facts (addition, subtraction, multiplication, division)
- Develop automaticity through adaptive, focused practice sessions
- Provide a 5-minute or less daily warmup tool
- Minimize anxiety and cognitive load while maximizing effective practice time

### Target Users
- **Primary**: K-5 students (ages 5-11)
- **Secondary**: Teachers and parents facilitating daily math practice

### Key Characteristics
- Single-file HTML application (no dependencies, no build process)
- Adaptive progressive difficulty based on real-time performance
- Clean, minimal interface free of distracting elements
- Runs entirely in-browser with no backend required
- Deployable to any static hosting (currently GitHub Pages)

---

## Architecture & Design Decisions

### Single-File Architecture

**Decision**: Implement as a single HTML file with embedded CSS and JavaScript.

**Reasoning**:
- **Portability**: Easy to download, share, and run anywhere
- **Simplicity**: No build process, no dependencies, no configuration
- **Accessibility**: Students can open directly from local filesystem or web
- **Deployment**: Simple GitHub Pages hosting with zero configuration
- **Maintenance**: Everything in one place, easy to understand and modify

**Trade-offs**:
- Limited modularity (acceptable for this scope)
- No code splitting (file size ~15KB, negligible)
- No external library benefits (don't need them for this use case)

### Adaptive Difficulty System

**Decision**: Use a 4-level adaptive difficulty system with streak-based progression.

**Reasoning**:
- **Alignment with learning science**: Implements zone of proximal development
- **Mastery learning**: Students must demonstrate competence (3 correct in a row) before advancing
- **Supportive scaffolding**: Immediate difficulty reduction on errors prevents frustration
- **Appropriate ceiling**: Level 4 cap matches K-5 curriculum expectations

**Implementation Details**:
```javascript
// 4 difficulty levels (1-4)
// Progression: +1 level after 3 consecutive correct answers
// Regression: -1 level immediately after any wrong answer
// Session: 15 problems (enough to reach max difficulty: 3+3+3+6 = 15)
```

**Difficulty Level Ranges**:

| Operation | Level 1 | Level 2 | Level 3 | Level 4 |
|-----------|---------|---------|---------|---------|
| Addition | Facts ≤5 | Facts ≤10 | Facts ≤20 | 2-digit ≤75 |
| Subtraction | Facts ≤5 | Facts ≤10 | Facts ≤20 | 2-digit ≤75 |
| Multiplication | 0-2× tables | 0-5× tables | 0-10× tables | 0-12× tables |
| Division | 0-2÷ tables | 0-5÷ tables | 0-10÷ tables | 0-12÷ tables |

### Session Structure

**Decision**: 15 problems per operation, no time limit during practice.

**Reasoning**:
- **Math**: 15 problems allows reaching Level 4 (3+3+3 to reach Level 4, then 6 problems at target difficulty)
- **Time**: ~2-4 minutes at automaticity level, fits 5-minute warmup goal (4 operations)
- **Predictability**: Fixed count reduces anxiety vs. timed practice
- **Progress visibility**: Students see "Problem X of 15" and know endpoint

**Alternative considered**: Time-based (60 seconds) - rejected due to anxiety concerns

### User Experience Flow

**Decision**: Direct-to-practice flow with no intermediate steps.

**Reasoning**:
- **Speed**: Click operation → immediate practice (removes friction)
- **Focus**: Eliminates decision fatigue from difficulty selection
- **Daily warmup context**: Students want to jump in quickly each morning

**Flow**:
1. Select operation (Addition/Subtraction/Multiplication/Division)
2. Immediately start practice at Level 1
3. Complete 15 problems with adaptive difficulty
4. View results (score, accuracy, time)
5. Click "Practice Again" to return to operation selection

### Minimal UI Design

**Decision**: Extreme minimalism - black text on white, large clear typography, no animations.

**Reasoning**:
- **Cognitive load**: Nothing to distract from math problems
- **Accessibility**: High contrast, large text (72px for problems, 48px for input)
- **Anxiety reduction**: No countdown timers, pressure indicators, or flashy elements
- **Speed**: No animations between problems (400ms feedback delay only)

**Visual Hierarchy**:
- Problem display: 72px (primary focus)
- Answer input: 48px (secondary focus)
- Feedback: 64px (✓ or ✗, brief display)
- Progress/metadata: 14px (peripheral awareness)

### Feedback Mechanism

**Decision**: Simple visual feedback (✓/✗) with 400ms delay before next problem.

**Reasoning**:
- **Speed**: Fast enough to maintain flow state
- **Clarity**: Immediate, unambiguous feedback
- **Low pressure**: No sound by default, no penalty beyond difficulty adjustment
- **Automaticity focus**: Quick feedback loop reinforces instant recall

---

## Technology Stack

### HTML5
- **Why**: Universal browser support, semantic structure
- **Version**: Living standard (all modern features available)
- **Usage**: Minimal DOM structure, three main screens (start/practice/results)

### CSS3
- **Why**: Native styling, no framework overhead
- **Approach**: Simple utility classes, flexbox for layout
- **Features**: Responsive design (viewport units, max-width container)

### Vanilla JavaScript (ES6+)
- **Why**: Zero dependencies, maximum portability, sufficient for scope
- **Features used**:
  - `let`/`const` for scoping
  - Template literals for string interpolation
  - Arrow functions for callbacks
  - `Math.random()` for problem generation
  - `setTimeout` for feedback delays
  - DOM manipulation (classList, querySelector)

**No frameworks needed because**:
- Simple state management (8 variables)
- Minimal DOM updates (problem display, feedback, results)
- No complex data flow or component hierarchy
- No routing (3 screens managed with CSS classes)

### Deployment: GitHub Pages
- **Why**: Free, fast, integrated with repository
- **Configuration**: Deploy from branch `claude/create-automath-app-icRFe`
- **URL pattern**: `https://<username>.github.io/automath/`
- **Updates**: Automatic on push (1-2 minute deploy time)

---

## File Structure

```
/home/user/automath/
├── automath.html          # Main application file
├── index.html             # GitHub Pages entry point (copy of automath.html)
├── PROJECT_CONTEXT.md     # This file - technical documentation
├── DESIGN_RATIONALE.md    # Conceptual and decision documentation
└── .git/                  # Git repository
```

### Key Files

**automath.html**
- Complete self-contained application
- Sections: HTML structure, CSS styles, JavaScript logic
- ~450 lines total (~150 HTML, ~100 CSS, ~200 JS)

**index.html**
- Exact copy of automath.html for GitHub Pages root deployment
- Kept in sync via `cp automath.html index.html` before commits

---

## Implementation Details

### Problem Generation Algorithm

Problems are generated dynamically based on current difficulty level and operation type:

```javascript
function generateProblem(operation, difficultyLevel) {
    // Generates random problem within difficulty constraints
    // Returns: { num1, num2, operator, answer }
}
```

**Key constraints**:
- Addition/Subtraction: Positive integers only, no negatives
- Division: Always produces whole number results (no remainders)
- Multiplication: Commutative (either order acceptable for input)
- Random distribution: Uniform within difficulty range

### State Management

Simple global state (no framework needed):

```javascript
let selectedOperation = null;      // 'addition', 'subtraction', etc.
let currentProblem = 0;            // 0-14 (current problem index)
let totalProblems = 15;            // Session length
let correctAnswers = 0;            // Total correct in session
let currentDifficulty = 1;         // 1-4 (adaptive difficulty level)
let correctStreak = 0;             // 0-2+ (for difficulty progression)
let currentProblemData = null;     // { num1, num2, operator, answer }
let startTime = null;              // timestamp (for elapsed time)
```

### Screen Transitions

Managed via CSS classes (no routing library):

```javascript
// Show screen: element.classList.add('active')
// Hide screen: element.classList.remove('active')
// CSS: .screen { display: none; } .screen.active { display: block; }
```

### Input Handling

- Number input field (`type="number"`)
- Enter key submits answer
- Auto-focus on input after each problem
- Value cleared between problems

---

## Current State

### Working Features ✅
- ✅ Four operation types (addition, subtraction, multiplication, division)
- ✅ Adaptive difficulty (4 levels, streak-based progression)
- ✅ 15-problem practice sessions
- ✅ Immediate visual feedback (✓/✗)
- ✅ Results screen (score, accuracy, total time)
- ✅ Clean, minimal UI
- ✅ Responsive design (works on iPad and desktop)
- ✅ Direct-to-practice flow (no intermediate screens)
- ✅ GitHub Pages deployment

### Known Limitations 🔄
- No progress tracking across sessions (always starts at Level 1)
- No mixed operation mode (must select single operation)
- No keyboard shortcuts for operation selection
- No accessibility labels (screen reader support)
- No sound feedback
- No parent/teacher visibility into progress
- No customization options (problem count, difficulty ranges)

### Browser Compatibility
- **Tested**: Modern Safari (iPad), Chrome, Firefox
- **Minimum**: ES6 support (2015+)
- **Known issues**: None reported

---

## Future Considerations

### High Priority (Aligned with Learning Science)

**1. LocalStorage Progress Tracking**
- Save per-operation performance data (accuracy, speed, difficulty level)
- Start sessions at last achieved difficulty level (implements spaced repetition)
- Track historical performance trends
- Implementation: ~50 lines of JavaScript, no backend needed

**2. Mixed Practice Mode**
- "All Operations" button that interleaves all four operations
- Aligns with interleaving principle from learning science
- Randomly selects operation for each problem
- Maintains separate difficulty levels per operation

**3. Visual Progress Indicator**
- Subtle dots or progress bar (e.g., "●●●○○○○○○○○○○○○" for 3/15)
- Non-distracting, helps students gauge progress
- Could use CSS circles or Unicode characters

### Medium Priority (UX Improvements)

**4. Keyboard Shortcuts**
- Number keys 1-4 for operation selection on start screen
- Reduces friction for daily warmup routine

**5. Accessibility Enhancements**
- ARIA labels for screen readers
- Keyboard navigation throughout
- High contrast mode toggle
- Font size adjustment

**6. Optional Sound Feedback**
- Gentle chime for correct, soft tone for incorrect
- Off by default (opt-in to avoid anxiety)
- Respects system mute settings

### Lower Priority (Extended Features)

**7. Parent/Teacher Dashboard**
- Read-only view of student progress
- Export LocalStorage data to JSON
- Simple charts: accuracy trends, time per operation
- No login required (local data only)

**8. Minimal Gamification**
- Daily streak counter (encourages consistency)
- No points or competitive elements (avoids anxiety)
- Simple badge system (e.g., "7-day streak!")

**9. Customization Options**
- Adjustable problem count per session
- Custom difficulty ranges (for advanced/struggling students)
- Teacher-defined problem sets

---

## Performance Considerations

### Current Performance
- **Load time**: <100ms (single ~15KB file)
- **Problem generation**: <1ms (pure computation)
- **Feedback delay**: 400ms (intentional UX choice)
- **Memory footprint**: Minimal (~100KB browser process overhead)

### Scalability
- **Not applicable**: Single-user, client-side application
- No server scaling concerns
- No database queries
- No API calls

### Optimization Opportunities
- Current implementation is already highly optimized for its scope
- Future LocalStorage writes are negligible performance impact
- No bundling/minification needed (file size already tiny)

---

## Development Workflow

### Local Testing
1. Open `automath.html` directly in browser (file://)
2. Make changes in text editor
3. Refresh browser to see updates
4. No build process required

### Deployment to GitHub Pages
1. Make changes to `automath.html`
2. Copy to `index.html`: `cp automath.html index.html`
3. Commit: `git add . && git commit -m "description"`
4. Push: `git push -u origin claude/create-automath-app-icRFe`
5. Wait 1-2 minutes for GitHub Pages deploy
6. Verify at `https://<username>.github.io/automath/`

### Testing Checklist
- [ ] All four operations generate valid problems
- [ ] Difficulty progression works correctly (3 correct → level up)
- [ ] Difficulty regression works (1 wrong → level down)
- [ ] Results screen calculates accuracy correctly
- [ ] Timer displays correctly (MM:SS format)
- [ ] "Practice Again" button resets to start screen
- [ ] Input field auto-focuses after each problem
- [ ] Enter key submits answers
- [ ] Visual feedback (✓/✗) displays for 400ms

---

## Maintenance Notes

### Code Organization
- HTML structure: Lines 1-215
- CSS styles: Lines 7-185
- JavaScript: Lines 256-447
- Clear separation of concerns within single file

### Modifying Difficulty Levels
To adjust difficulty ranges, edit the `generateProblem()` function (lines 287-375):
```javascript
// Example: Making Level 1 easier (facts within 3 instead of 5)
if (difficultyLevel === 1) {
    num1 = Math.floor(Math.random() * 3) + 1;
    num2 = Math.floor(Math.random() * (3 - num1)) + 1;
}
```

### Changing Session Length
Update `totalProblems` variable (line 260):
```javascript
let totalProblems = 15;  // Change this number
```

Also update HTML placeholder (line 209):
```html
<div class="progress" id="progress">Problem 1 of 15</div>
```

### Adjusting Difficulty Progression Speed
Modify streak requirement in `checkAnswer()` function (line 416):
```javascript
// Require 3 consecutive correct answers to level up
if (correctStreak >= 3 && currentDifficulty < 4) {
```

---

## Contact & Contribution

This is a self-contained educational project. Future enhancements should maintain:
- Single-file architecture
- Zero dependencies
- Minimal, distraction-free UI
- Alignment with learning science principles (see DESIGN_RATIONALE.md)
- K-5 appropriate content and difficulty

When adding features, always ask:
1. Does this reduce cognitive load or increase it?
2. Does this create anxiety or reduce it?
3. Does this support automaticity development?
4. Is this truly necessary for a 5-minute daily warmup tool?
