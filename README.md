# Adaptive Testing for Student Knowledge Levels
 _Figure: Overview of an adaptive testing process. The system selects a question for the student, scores the answer, and loops until a stopping condition is reached (e.g. enough precision or max questions)._ Adaptive testing is a well-established method to estimate a student’s ability by adjusting question difficulty on the fly. In an adaptive test, if a student answers correctly the next question becomes harder; if they answer incorrectly, the next question becomes easier. This loop continues – selecting and scoring items – until the test ends. In effect, the student’s knowledge level is identified as the highest difficulty at which they can consistently answer questions correctly.

## Stepwise Progressive Algorithm
A simple way to locate the student’s level is a _stepwise search_ starting from the easiest level and moving upward. For example:

1. **Start at level 0.** Ask a level-0 question.
2. **Check the answer.**
    - If the student answers correctly, **advance one level** (e.g. from 0 to 1).
    - If the student answers incorrectly, **drop one level** (but not below 0).
3. **Repeat.** Ask the next question at the new level and repeat step 2.
This method “homes in” on the boundary between the student’s mastery and non-mastery. By increasing the level when the student succeeds and decreasing when they fail, the algorithm effectively performs a search for the highest level the student can handle. In practice, to avoid misclassification from a single mistake or lucky guess, one usually requires multiple confirmations at each level. For example, you might insist on _two consecutive correct answers_ before moving up a level, and _two consecutive wrong answers_ before moving down. This way one mistake doesn’t immediately reset the level.

## Staircase (Consecutive-Response) Rules
A more formal version of the above idea is the _staircase method_, which is common in adaptive testing literature. In a staircase algorithm you adjust difficulty only after a fixed streak of successes or failures. For instance, a **“two-up, two-down” rule** works as follows:

- **Upward move:** If the student answers **two consecutive questions correctly** at the current level, then raise the level by one.
- **Downward move:** If the student answers **two consecutive questions incorrectly**, then lower the level by one (down to a minimum of 0).
- **Maintain:** If neither streak condition is met (for example, if answers alternate), stay at the same level.
This approach ensures that difficulty only changes when performance is clearly above or below mastery. (Some systems use a “three-up, three-down” rule, requiring three correct in a row to move up and three wrong to move down.) As an example, in the scenario described, the student got level-2 questions wrong, then right, then wrong again. Under a 2-up-2-down rule, this does not produce two straight failures, so the system would be hesitant to keep moving down from level 2. After these mixed results at level 2, the algorithm correctly settles on level 1 as the student’s stable level, consistent with their ability to answer level-1 questions reliably but not level-2.

## Binary-Search Approach
Another strategy, given the levels are in a known order, is a _binary search_ over levels. The procedure is:

- Initialize a range  = .
- Ask a question at the middle level, mid = ⌊(low+high)/2⌋.
- If the student answers correctly, set low = mid + 1 (their ability is at least mid).
- If incorrect, set high = mid – 1.
- Repeat until low > high. The student’s level is then inferred from the last successful question or the final range.
This binary-style search can quickly narrow down the level in about log₂(11) ≈ 4 questions if answers are consistent. In practice, because responses can be noisy, one would still verify the final candidate levels with extra questions (e.g. ensure multiple correct at that level). Nevertheless, the binary method minimizes the number of questions by always splitting the remaining levels in half and discarding the irrelevant half based on the answer.

## Algorithm Implementation (Pseudocode)
Putting these ideas together, a practical algorithm using a simple up/down rule can be implemented with counters. For example, using a 2-up/2-down rule in pseudocode:

```
current_level = 0
correct_streak = 0
wrong_streak = 0

for i in 1..N_questions:
    ask_question(current_level)
    if answer == correct:
        correct_streak += 1
        wrong_streak = 0
    else:
        wrong_streak += 1
        correct_streak = 0

    # If two correct in a row, move up
    if correct_streak >= 2 and current_level < 10:
        current_level += 1
        correct_streak = 0
    # If two wrong in a row, move down
    if wrong_streak >= 2 and current_level > 0:
        current_level -= 1
        wrong_streak = 0

# After N questions or convergence:
return current_level
```
In this code, `current_level` tracks the tested level, and we only change it when two consecutive answers support the move. You can adjust the thresholds (e.g. use `>= 3` for a 3-up-3-down rule) or change how many questions _N_ to ask in total.

## Key Considerations
- **Question Bank:** Ensure you have at least one (or ideally several) question per level. To verify level mastery, you may need to ask more than one question at that level.
- **Termination:** Decide when to stop. For a fixed-question test you stop after _N_ questions. Otherwise you could stop once the student has clearly demonstrated a level (e.g. after two failures at a higher level) or when a confidence measure (like a psychometric precision) is achieved.
- **Robustness:** Requiring multiple correct or incorrect answers at a level guards against lucky guesses or careless mistakes. For example, the 2-up-2-down (or 3-up-3-down) rules mentioned above are standard ways to add “tolerance” and ensure the result is stable.
- **Final Level:** In the end, report the highest level at which the student consistently answered correctly. In the running example, the student consistently passed level 1 but not level 2, so level 1 is the result.
**Summary:** The student’s knowledge can be estimated by an adaptive algorithm that starts at a low level and moves up or down based on answers. A practical solution is to require a short streak of correct answers to advance and a streak of wrong answers to retreat. This effectively “homes in” on the student’s true level with relatively few questions, ensuring that level 0..10 is correctly determined.
