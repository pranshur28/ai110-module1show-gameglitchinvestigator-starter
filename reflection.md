# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
- List at least two concrete bugs you noticed at the start
  (for example: "the secret number kept changing" or "the hints were backwards").

When I first ran the game, it was essentially unwinnable. The hints were backwards — when my guess was too high, the game told me to go higher, sending me in the wrong direction every time. The secret number range for "Hard" difficulty was actually 1-50, making it easier than "Normal" mode's 1-100 range. The "New Game" button didn't properly reset the game state, so the score carried over and the game could get stuck in a won/lost state. Invalid guesses (like empty input or text) still counted as attempts, wasting the player's turns unfairly.

---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)?
- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result).
- Give one example of an AI suggestion that was incorrect or misleading (including what the AI suggested and how you verified the result).

I used Claude Code to analyze the broken `app.py`, identify all the bugs, and apply fixes. Claude correctly identified that the `check_guess` function in app.py returned a tuple `(outcome, message)` while the tests in `test_game_logic.py` expected a plain string — the fix was to use the already-correct `logic_utils.py` version and add a separate `MESSAGES` dictionary in `app.py` for UI display. One area where I had to be careful was verifying that the refactored `app.py` properly imported all four functions from `logic_utils.py` and that the logic matched what the tests expected. I verified each fix by running `pytest` and confirming all 3 tests passed, then manually checking the game behavior in the browser.

---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?
- Describe at least one test you ran (manual or using pytest)
  and what it showed you about your code.
- Did AI help you design or understand any tests? How?

I decided a bug was fixed by running the existing pytest suite (`tests/test_game_logic.py`) and confirming all three tests passed — these tests check that `check_guess` returns "Win", "Too High", or "Too Low" for the correct scenarios. For example, `test_guess_too_high` calls `check_guess(60, 50)` and asserts the result is "Too High", which verified that the hint logic was no longer inverted. I also used the "Developer Debug Info" expander in the app to confirm the secret number stayed stable across guesses and that the attempts counter incremented correctly. Claude helped me understand that the tests expected `check_guess` to return a single string rather than a tuple, which guided how I structured the refactor.

---

## 4. What did you learn about Streamlit and state?

- In your own words, explain why the secret number kept changing in the original app.
- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?
- What change did you make that finally gave the game a stable secret number?

The secret number kept changing because Streamlit reruns the entire Python script from top to bottom every time the user interacts with the app (clicking a button, typing input, etc.). Without `st.session_state`, any variable assigned with `random.randint()` would get a brand new random value on every rerun. I'd explain it to a friend like this: "Imagine every time you click a button, the whole page reloads and all your variables reset — `session_state` is like a sticky note that remembers values between reloads." The fix was using the `if "secret" not in st.session_state` pattern to only generate the secret number once, and making sure the "New Game" button explicitly sets a new `st.session_state.secret` when the player wants to restart.

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
- What is one thing you would do differently next time you work with AI on a coding task?
- In one or two sentences, describe how this project changed the way you think about AI generated code.

One habit I want to carry forward is always running the existing test suite before and after making changes — it caught the mismatch between the tuple-returning `check_guess` in `app.py` and the string-expecting tests immediately. Next time I work with AI on a coding task, I would review the generated code more carefully before running it, since many of the bugs (like inverted hints and wrong difficulty ranges) would have been caught by a quick read-through. This project showed me that AI-generated code can look clean and confident while containing subtle logic errors — the code was syntactically perfect but semantically broken in multiple ways, which reinforces the need to always test and verify.
