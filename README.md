# Lab 8 walkthrough: CI and test-driven development

This guide follows [`lab-08-description.md`](lab-08-description.md) using the Android project in `lab8/`. It explains the walkthrough, the common failure points, and how to complete the `train()` participation exercise. The repository root and the Android project root are different directories: open `lab8/` in Android Studio, but run Git commands from the repository root.

## 1. Set up the fork and check CI

Fork the lab repository and confirm that the fork is public, as the handout requires. Clone **your fork**, then open its `lab8/` folder in Android Studio and let Gradle sync. On the fork's **Actions** tab, enable workflows if GitHub asks. The workflow is in `.github/workflows/ci.yml`; it runs on pushes and builds the project from `lab8/` with `./gradlew build --configuration-cache`.

Make a harmless change, commit it, and push it to your fork. The resulting Actions run should show the baseline build and existing unit tests passing before you start TDD. Open the run's logs if it fails; a yellow icon means the run is still in progress, not that it passed.

**Starter pitfall:** the original `ci.yml` selected Java 11, but this repository's `gradle/gradle-daemon-jvm.properties` requests Java 21 and its Gradle wrapper is 9.4.1. The workflow in this repository now sets up Java 21. If you copied the earlier workflow into your fork, change `java-version` to `'21'` before interpreting a CI failure as a test failure. Keep the `working-directory: ./lab8` entries: the Gradle wrapper is inside that folder, not at the repository root.

## 2. Walk through `restoreMana()` with red and green runs

The starter `Wizard` is in `lab8/app/src/main/java/com/example/lab_8/Wizard.kt`. Its tests are in `lab8/app/src/test/java/com/example/lab_8/WizardTest.kt`. These are **local JUnit tests** under `src/test`, so `gradlew test` or the CI build can run them without an emulator. The separate `src/androidTest` folder is for instrumented device tests.

Add this method **inside** the `Wizard` class to make the tests compile while leaving its behavior unfinished:

```kotlin
fun restoreMana(amount: Int) { }
```

In `WizardTest.kt`, add the two tests from the handout. Each test starts with a fresh wizard because `@Before setUp()` creates one with `mana = 30`. One test restores 80 and expects the cap of 100; the other restores 40 and expects 70. Commit and push the skeleton and tests **before** writing the implementation. The Actions run should be red because the assertions fail, not because Kotlin fails to compile or Gradle cannot start. Preserve that failed run and commit in the fork's history; this is evidence of the test-first step.

Then implement the handout's method:

```kotlin
fun restoreMana(amount: Int) {
    mana = minOf(100, mana + amount)
}
```

Commit and push again. Check the new Actions run until it finishes green. `restoreMana()` belongs in `Wizard.kt`, not in the test class. If `mana` does not change, check that the method assigns the `minOf` result back to the property. The handout does not specify how negative restoration amounts should behave, so the walkthrough only tests the two positive cases it gives.

> **Walkthrough ends here.** The steps below implement the participation exercise. Keep its red and green CI runs as separate commits and pushes.

## 3. Participation exercise: specify `train()` before implementing it

The required behavior is: with at least five mana, one call increases `spellPower` by one and decreases `mana` by five; with fewer than five mana, neither property changes. The method takes no parameters and returns no value.

First add only a compilable skeleton inside `Wizard`:

```kotlin
fun train() { }
```

Then add **two** tests to `WizardTest.kt`. These examples check both affected properties, including the boundary at exactly five mana:

```kotlin
@Test
fun train_withFiveMana_increasesPowerAndSpendsMana() {
    evilWizard.mana = 5

    evilWizard.train()

    assertEquals(11, evilWizard.spellPower)
    assertEquals(0, evilWizard.mana)
}

@Test
fun train_withFourMana_leavesPowerAndManaUnchanged() {
    evilWizard.mana = 4

    evilWizard.train()

    assertEquals(10, evilWizard.spellPower)
    assertEquals(4, evilWizard.mana)
}
```

The first test must fail with the empty method: it expects power 11 and mana 0, while the skeleton leaves them at 10 and 5. The second test should already pass. That is a useful red run; the goal is to expose the missing behavior, not to make every test fail. Commit and push the skeleton and tests now, and confirm that the **test assertion** fails in Actions. Do not add the working implementation to this commit; otherwise the required red CI run will never exist.

After the red run, implement the smallest method that meets both tests:

```kotlin
fun train() {
    if (mana >= 5) {
        mana -= 5
        spellPower += 1
    }
}
```

Commit and push this change separately. Wait for the new Actions run to pass. A check of only `spellPower` could miss a method that grants power for free; a check of only `mana` could miss a method that spends mana without training. The `>= 5` comparison matters because a wizard with exactly five mana must be able to train. When mana is four, both values must remain unchanged.

## 4. Diagnose common failures

| Symptom | Check |
| --- | --- |
| CI fails before tests start | Check the Java version in `ci.yml`, the `lab8/` working directory, Gradle sync, and the first error in the Actions log. A setup failure is not the TDD red run. |
| `Unresolved reference: train` or `restoreMana` | Put the skeleton in the `Wizard` class first, with exactly the method name and signature the tests call. |
| The tests pass before implementation | Verify that the success test asserts **both** new values and starts at the intended mana. Do not mistake an all-green skeleton for proof of TDD. |
| The red run is a compilation error | Fix imports, braces, or method placement so tests compile and fail on an assertion. `assertEquals` comes from `org.junit.Assert`. |
| Local tests pass but GitHub is red | Open the specific failed Actions job and compare its commit SHA with your latest push. A prior red run remains red after a later green run. |
| No Actions run appears | Confirm Actions are enabled on **your fork**, that the push reached its branch, and that you are viewing the fork rather than the course repository. |

From `lab8/`, run `./gradlew test` on macOS/Linux or `.\gradlew.bat test` in Windows PowerShell for a quick local test. Use the fork's Actions tab for the required CI evidence. If you run Git from `lab8/`, Git still finds the parent repository, but use repository-root paths when staging files to avoid confusion.

## 5. Submission checklist

- Confirm the fork is accessible as required by the handout and that the history shows the `train()` tests failing before the implementation commit, followed by a passing run.
- Replace the student detail placeholders below with **your** full name and CCID. Record resources and verbal collaborators accurately; use `N/A` where appropriate.
- Put your full name in `LICENSE.md`, then submit the link to your fork on Canvas. The course handout and the starter license give conflicting public-repository instructions; follow the course staff's current direction before submitting.

## Student details

- **Full name:** `<Enter name>`
- **CCID:** `<Enter ccid>`

## References and resources

- `lab-08-description.md` and the starter project in this repository.
- Add any other resources used, or write `N/A`.

## Verbal collaboration

- Add collaborators' names and CCIDs, or write `N/A`.
