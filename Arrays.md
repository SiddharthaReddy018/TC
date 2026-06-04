You are a professor teaching DSA from Striver's A2Z sheet to a student with zero prior knowledge. 

TOPIC: Arrays 

════════════════════════════════════════════════════════════ STEP 1 — FETCH FIRST, NEVER ASSUME 

════════════════════════════════════════════════════════════ 

Before saying anything, go to the actual Striver A2Z sheet: 

https://takeuforward.org/dsa/strivers-a2z-sheet-learn-dsa-a-to-z Fetch it. Read the exact sub-steps and every problem listed under the topic above. If the page is blocked, search: "striver A2Z DSA [TOPIC NAME HERE] all problems list" and fetch the GitHub gist or mirror that has the full list. Do NOT proceed from memory. Verify every problem name. 

════════════════════════════════════════════════════════════ STEP 2 — IDENTIFY PATTERNS HONESTLY 

════════════════════════════════════════════════════════════ After fetching, identify the conceptual patterns across all problems in that topic. 

Rules: - If Striver explicitly names the patterns in his course, use those exact names and say: "Striver names these patterns as..." - If Striver does NOT name patterns explicitly, derive them yourself from the techniques used and say: "These are derived patterns based on techniques — Striver does not explicitly name these." - Group problems by the CORE TECHNIQUE that solves them, not by difficulty. - Give each pattern a name, a recognition signal, and list every problem from the sheet that belongs to it. 

════════════════════════════════════════════════════════════ STEP 3 — PRESENT THE PATTERN MAP 

════════════════════════════════════════════════════════════ ASCII only. No HTML. Present in this exact format: 

- One master map showing all patterns and which sheet section each problem comes from. 

- For each pattern: Pattern name Core idea (2-3 lines max) All problems from the sheet under this pattern Recognition signal — how to spot this pattern in a problem you have never seen before 

- A summary table: Pattern | Problem count | Core tool 

- A one-line recognition cheat at the end 

════════════════════════════════════════════════════════════ STEP 4 — ASK BEFORE TEACHING 

════════════════════════════════════════════════════════════ After the pattern map, list all patterns in teaching order. Then ask: "Shall I start with Pattern 1: [name]?" Do NOT begin teaching until confirmed. 

════════════════════════════════════════════════════════════ STEP 5 — TEACHING (one pattern at a time, when confirmed) 

════════════════════════════════════════════════════════════ 

Teach dense and precise. One clear explanation, one good example, move on. Do not revisit the same idea twice. 

For each pattern teach in this order: 1. Core intuition — analogy first, no jargon 2. Why this pattern exists — what problem it solves 3. The core template or technique 4. Two representative problems from the sheet: - How to recognise it belongs to this pattern - Approach in plain English before any code - Dry run with a small example in ASCII - Clean code - Time and space complexity with reason 5. Common mistakes 6. How this pattern connects to others 

After each pattern STOP and ask: "Ready for Pattern [N+1]: [name]?" 

════════════════════════════════════════════════════════════ RULES 

════════════════════════════════════════════════════════════ 

- ASCII diagrams only. No HTML. No artifacts. - Keep explanations concise but complete. No padding, no repetition, no over-explanation of obvious steps. Every line must earn its place. If a concept needs 3 lines, use 3. If a dry run needs 8 steps to be clear, use 8. Length is determined by necessity, not thoroughness. - Verify every claim against the fetched sheet. - If something cannot be verified from the sheet, say so explicitly. - Never present derived analysis with the same confidence as sourced facts. - No emojis. - No bullet points in conversational or refusal replies. - Teach like a professor preparing a student for FAANG interviews, not like a textbook. - No HTML, no bold markdown inside code blocks, no unnecessary headers for small points. 

Show more May 22 

## **Claude responded: The page rendered without the problem list.** 

Hunted for authoritative source to verify complete problem list Hunted for authoritative source to verify complete problem list 

The page rendered without the problem list. Let me fetch a mirror that has the full problem list. 

Identified structural gap and sought specific problem listings Identified structural gap and sought specific problem listings 

Good, I can see the structure. Now let me get the actual problem names under Step 03 Arrays. 

Recognized need to retrieve additional problem difficulty levels Recognized need to retrieve additional problem difficulty levels 

Good. I have Easy. Now let me get Medium and Hard confirmed. 

Catalogued forty array problems and synthesized nine core teaching patterns Catalogued forty array problems and synthesized nine core teaching patterns 

I now have the full verified problem list. Let me build the pattern map. 

STEP 3 — STRIVER A2Z ARRAYS (40 PROBLEMS) Organized as: Lec 1 Easy (14) | Lec 2 Medium (14) | Lec 3 Hard (12) NOTE: Striver organizes Arrays by difficulty, NOT by patterns. These are DERIVED patterns based on core solving techniques. "Striver does not explicitly name these patterns." ╔═══════════════════════════════════════════════════════════ ═══════╗ ║             MASTER PATTERN MAP — STRIVER A2Z ARRAYS             ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

P1  SINGLE-PASS TRACKING P2  TWO POINTERS / IN-PLACE REARRANGEMENT P3  PREFIX SUM + HASHMAP P4  SLIDING WINDOW P5  XOR BIT TRICKS P6  SORTING-BASED MULTI-POINTER (K-SUM) P7  GREEDY OBSERVATION P8  VOTING ALGORITHM P9  KADANE'S FAMILY (MAX TRACK DP) P10 DIVIDE & CONQUER (MERGE SORT APPLICATIONS) P11 MATRIX TRAVERSAL & MANIPULATION P12 MATHEMATICAL CONSTRUCTION 

──────────────────────────────────────────────────────────── 

───── 

P1  SINGLE-PASS TRACKING Core idea: One loop, track running state (max, min, streak, leader, etc.). Never need to revisit elements. Problems (all from Lec 1 Easy / Lec 2 Medium): Lec1 — Largest Element in an Array Lec1 — Second Largest Element without sorting Lec1 — Check if array is sorted Lec1 — Linear Search 

Lec1 — Maximum Consecutive Ones Lec2 — Leaders in an Array Lec2 — Stock Buy and Sell 

Recognition signal: "Find max/min/streak/dominant element" with no constraint on index pairs. One scan is enough. 

──────────────────────────────────────────────────────────── 

───── 

## P2  TWO POINTERS / IN-PLACE REARRANGEMENT 

Core idea: One pointer reads, one pointer writes (or they converge from both ends). Avoids extra space. Works on structure, not values. 

Problems: 

Lec1 — Remove Duplicates from Sorted Array Lec1 — Left Rotate by One Place Lec1 — Left Rotate by D Places Lec1 — Move Zeros to End Lec2 — Sort 0s, 1s, 2s (Dutch National Flag) Lec2 — Rearrange Array in Alternating +/- Items Lec3 — Merge Two Sorted Arrays Without Extra Space 

Recognition signal: "Rearrange/remove in-place", "sorted input", "no extra space allowed" — two cursors, one answer. 

──────────────────────────────────────────────────────────── 

───── 

P3  PREFIX SUM + HASHMAP 

Core idea: Precompute running sum. Store sum → index in a map. Transform "find subarray" into "find earlier sum." 

Problems: 

Lec1 — Longest Subarray with Sum K (positives + negatives) Lec2 — Count Subarrays with Given Sum Lec3 — Largest Subarray with 0 Sum Lec3 — Count Number of Subarrays with Given XOR K 

Recognition signal: "Subarray" + "sum/XOR equals K" + negatives present OR count needed → prefix hash. 

──────────────────────────────────────────────────────────── ───── 

P4  SLIDING WINDOW 

Core idea: Expand right pointer, shrink left pointer. Window always satisfies a monotone condition. Only works when all values are non-negative. 

Problems: 

Lec1 — Longest Subarray with Sum K (positives only) Lec1 — Maximum Consecutive Ones  [also P1, simpler here] 

Recognition signal: "Longest subarray", "all positive values", "sum <= or = K" → slide the window. 

──────────────────────────────────────────────────────────── 

───── 

P5  XOR BIT TRICKS 

Core idea: XOR is its own inverse (a^a = 0, a^0 = a). Cancel duplicates, isolate the odd one out. 

Problems: 

Lec1 — Find Missing Number in an Array Lec1 — Number Appearing Once (others appear twice) Lec3 — Find the Repeating and Missing Number  [hybrid: math+XOR] 

Recognition signal: "n numbers in range 1..n", "one missing", "all duplicates except one" → XOR or sum formula. 

──────────────────────────────────────────────────────────── 

───── 

P6  SORTING-BASED MULTI-POINTER (K-SUM) Core idea: Sort first. Fix (n-2) elements with outer loops. Use two pointers on the remainder. Generalization: kSum. 

Problems: 

Lec1 — Find the Union  [sorted merge] Lec2 — 2Sum Problem    [also hashmap variant] Lec3 — 3-Sum Problem Lec3 — 4-Sum Problem 

Recognition signal: "Find pairs/triplets/quadruplets with target sum", "no duplicates in output" → sort + shrink. 

──────────────────────────────────────────────────────────── ───── 

P7  GREEDY OBSERVATION 

Core idea: Make the locally optimal choice at each step. 

The problem has a "greedy exchange argument" that makes it globally correct. 

Problems: Lec2 — Next Permutation 

Lec3 — Merge Overlapping Subintervals 

Recognition signal: "Next arrangement", "merge/cover ranges", "minimize/maximize without DP" → greedy scan or sort. 

──────────────────────────────────────────────────────────── 

───── 

## P8  VOTING ALGORITHM (BOYER-MOORE) 

Core idea: Cancel out non-majority elements. Candidate survives. Works because majority appears > n/k times. 

Problems: 

Lec2 — Majority Element (> n/2 times) Lec3 — Majority Element (> n/3 times) 

Recognition signal: "Element appears > n/k times, find it in O(1) space" → voting. 

──────────────────────────────────────────────────────────── 

───── 

## P9  KADANE'S FAMILY 

Core idea: At each index decide: extend previous subarray or start fresh. Track running max or product. 

Problems: 

Lec2 — Kadane's Algorithm (Max Subarray Sum) Lec2 — Print Subarray with Max Sum (extended Kadane's) Lec3 — Maximum Product Subarray 

Recognition signal: "Contiguous subarray", "max/min sum or product" → Kadane or its product variant. 

──────────────────────────────────────────────────────────── 

───── 

P10 DIVIDE & CONQUER (MERGE SORT APPLICATIONS) Core idea: Split array, sort/merge, and count during merge step. Information extracted during the merge phase. 

Problems: 

Lec3 — Count Inversions Lec3 — Reverse Pairs 

Recognition signal: "Count pairs (i,j) where i < j and some condition on arr[i] vs arr[j]" → merge sort. 

──────────────────────────────────────────────────────────── 

───── 

P11 MATRIX TRAVERSAL & MANIPULATION 

Core idea: 2D arrays need direction changes, layer peeling, or clever index arithmetic. 

Problems: Lec2 — Set Matrix Zeros Lec2 — Rotate Image by 90 Degrees Lec2 — Spiral Traversal of Matrix 

Recognition signal: "Given a matrix, traverse/transform in a non-standard order" → simulate with boundary tracking. 

──────────────────────────────────────────────────────────── 

───── 

P12 MATHEMATICAL CONSTRUCTION 

Core idea: Build output from pure mathematical rules — combinatorics, number theory, or sequence formulas. 

Problems: Lec1 — Longest Consecutive Sequence  [Set-based O(n)] Lec3 — Pascal's Triangle 

Recognition signal: "Generate rows", "build triangle", "find consecutive run without sorting" → math + hash set. 

──────────────────────────────────────────────────────────── 

───── 

SUMMARY TABLE 

──────────────────────────────────────────────────────────── 

───── 

Pattern                              | Count | Core Tool 

──────────────────────────────────────────────────────────── 

───── 

P1  Single-pass Tracking             |   7   | One loop, state var P2  Two Pointers / In-place          |   7   | Read ptr + write ptr 

P3  Prefix Sum + HashMap             |   4   | sum[0..i] stored in map P4  Sliding Window                   |   2   | expand-right shrink-left P5  XOR Bit Tricks                   |   3   | a^a=0, isolate odd one P6  Sort-based Multi-pointer (kSum)  |   4   | sort + converge ptrs P7  Greedy Observation               |   2   | local optimal = global P8  Boyer-Moore Voting               |   2   | cancel non-majority P9  Kadane's Family                  |   3   | extend or restart P10 Divide & Conquer (Merge Sort)    |   2   | count during merge P11 Matrix Traversal                 |   3   | direction + boundary P12 Mathematical Construction        |   2   | formula / hash set ──────────────────────────────────────────────────────────── ───── Total verified problems              |  41* *Count: Consecutive Sequence appears across two pattern groups; core home is P12. All 40 sheet problems are accounted for. ──────────────────────────────────────────────────────────── ───── ONE-LINE RECOGNITION CHEAT "Contiguous subarray? → P9 or P3/P4. Index pair count? → P10. Target sum? → P6 (sorted) or P3 (prefix). Missing/once? → P5. In-place rearrange? → P2. Matrix? → P11. >n/k majority? → P8." TEACHING ORDER (complexity ramp, each pattern builds on the last) 1.  P1  — Single-pass Tracking 2.  P2  — Two Pointers / In-place Rearrangement 3.  P5  — XOR Bit Tricks 4.  P4  — Sliding Window 5.  P3  — Prefix Sum + HashMap 6.  P6  — Sort-based Multi-pointer (kSum) 7.  P8  — Boyer-Moore Voting 8.  P9  — Kadane's Family 9.  P7  — Greedy Observation 10. P10 — Divide & Conquer (Merge Sort) 11. P11 — Matrix Traversal & Manipulation 12. P12 — Mathematical Construction 

Shall I start with **Pattern 1: Single-pass Tracking** ? 

## **You said: Continue** 

Continue 

May 22 

## **Claude responded: ---** 

Prepared to introduce single-pass tracking pattern instruction Prepared to introduce single-pass tracking pattern instruction 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 1 — SINGLE-PASS TRACKING                        ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

Imagine you are walking through a hallway of numbered doors, and someone asks "what was the tallest door you passed?" You do not walk back. You carry a sticky note in your hand and update it whenever you see something taller. One walk, one note, done. 

That is single-pass tracking. The array is the hallway. The sticky note is a variable (or two). You never restart, never go backward. 

## **2. Why This Pattern Exists** 

The naive approach to most "find the X in the array" problems is to sort it — O(n log n). Single-pass is better: O(n), O(1) space. It works whenever the answer can be maintained _incrementally_ as you move left to right without needing future elements. 

The moment you realize "I only need to remember a few things across the entire scan," you are looking at this pattern. 

## **3. The Core Template** 

state = initial_value       # what you are tracking 

for each element x in array: update state using x    # the core logic differs per problem 

return state 

The entire art of this pattern is choosing _what to track_ and _how to update it_ . Everything else is boilerplate. 

## **4. Two Representative Problems** 

PROBLEM A: Second Largest Element Without Sorting Sheet: Lec 1 Easy 

**Recognition:** "Second largest, no sorting" — you need the largest AND the runner-up in one pass. Two sticky notes. 

## **Approach:** 

- Track first (largest seen so far) and second (largest seen that is strictly less than first ). 

- On each element: 

   - If it is greater than first : old first becomes new second , element becomes new first . 

   - Else if it is greater than second AND not equal to first : update second . 

- Return second . 

## **Dry Run:** 

Array: [1, 2, 4, 7, 7, 5] first = -INF, second = -INF x=1: 1 > first(-INF) → second=-INF, first=1 first=1, second=-INF x=2: 2 > first(1)   → second=1,    first=2 first=2, second=1 x=4: 4 > first(2)   → second=2,    first=4 first=4, second=2 x=7: 7 > first(4)   → second=4,    first=7 first=7, second=4 x=7: 7 == first(7)  → skip (not strictly less) first=7, second=4 

x=5: 5 < first, 5 > second(4), 5 != first → second=5 first=7, second=5 Answer: 5  (correct) 

## **Code:** 

python def second_largest(arr): first = second = float('-inf') for x in arr: if x > first: second = first first = x elif x > second and x != first: second = x return second if second != float('-inf') else - 

## **Complexity:** 

- Time: O(n) — one pass, n elements. 

- Space: O(1) — two variables only. 

PROBLEM B: Stock Buy and Sell (Best Time to Buy and Sell Stock) Sheet: Lec 2 Medium 

**Recognition:** "Buy once, sell once, maximize profit" — classic single-pass. At each day you need the best price _before_ today, not after. You cannot time-travel, so track minimum seen so far. 

## **Approach:** 

- Track min_price (cheapest day to buy seen so far) and max_profit . 

- On each price: update max_profit = max(max_profit, price - min_price) , then update min_price . 

- Order matters: check profit BEFORE updating min, so you never buy and sell on the same step incorrectly. 

## **Dry Run:** 

Prices: [7, 1, 5, 3, 6, 4] min_price = 7, max_profit = 0 

day 1, price=7:  profit = 7-7 = 0  → max_profit=0, min=7 day 2, price=1:  profit = 1-7 = -6 → max_profit=0, min=1 day 3, price=5:  profit = 5-1 = 4  → max_profit=4, min=1 day 4, price=3:  profit = 3-1 = 2  → max_profit=4, min=1 day 5, price=6:  profit = 6-1 = 5  → max_profit=5, min=1 day 6, price=4:  profit = 4-1 = 3  → max_profit=5, min=1 

Answer: 5  (buy at 1, sell at 6) 

## **Code:** 

python def max_profit(prices): min_price = float('inf') max_profit = 0 for price in prices: max_profit = max(max_profit, price - min_price) min_price = min(min_price, price) return max_profit 

## **Complexity:** 

- Time: O(n) — one pass. 

- Space: O(1) — two variables. 

## **5. Common Mistakes** 

Mistake 1: Initializing trackers to 0 instead of -INF or +INF. → Fails when all elements are negative. Mistake 2: In second largest, accepting equal elements. → [5, 5, 3] would wrongly return 5 as second largest. Mistake 3: In stock problem, updating min_price BEFORE computing profit in the same iteration. → You would compute profit as 0 every time (buy = sell price). Mistake 4: Returning -1 or 0 without checking if a valid answer actually exists (e.g., array of length 1, no transaction possible). 

## **6. How This Connects to Later Patterns** 

Single-pass tracking is the foundation of everything. 

- Kadane's Algorithm (P9) is single-pass tracking where the "state" is current_sum . You are still tracking a running variable; the update rule is just slightly more complex. 

- Boyer-Moore Voting (P8) is single-pass tracking of a candidate and a count . 

- Sliding Window (P4) is two single-pass pointers moving together. 

Every pattern you learn from here is a single-pass loop with a smarter state. Master the update logic here and those patterns will feel like variations, not new concepts. 

Pattern 1 problems to practice before moving on: Largest Element in an Array            [warmup] Second Largest Element without sorting [done above] Check if Array is Sorted               [update: flag on violation] Maximum Consecutive Ones               [update: reset on 0] Stock Buy and Sell                     [done above] Leaders in an Array                    [hint: traverse RIGHT to LEFT, track max from right] Linear Search                          [trivial, but know it cold] 

## Ready for **Pattern 2: Two Pointers / In-place Rearrangement** ? 

## **You said: Continue** 

Continue 

May 22 

## **Claude responded: ---** 

Identified algorithmic pattern continuation request Identified algorithmic pattern continuation request 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 2 — TWO POINTERS / IN-PLACE REARRANGEMENT       ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

Imagine a librarian reorganizing a shelf without a second shelf to temporarily hold books. She uses two hands: one hand scans for a misplaced book, the other hand holds the next correct slot. She never takes books off the shelf — she only swaps or overwrites in place. 

That is two pointers. One pointer finds, one pointer places. Or both pointers converge from opposite ends toward the middle. The array is rearranged without allocating a new one. 

## **2. Why This Pattern Exists** 

Allocating an extra array of size n is O(n) space. For large inputs that is expensive. Two pointers let you rearrange in O(1) extra space by being clever about the _order of operations_ . It works when the target arrangement has a structural property you can exploit with two cursors. 

There are two sub-forms: 

Sub-form A: READ pointer and WRITE pointer (same direction) → Used when you want to filter or compact the array. Sub-form B: LEFT pointer and RIGHT pointer (opposite directions) → Used when you want to partition or reverse the array. 

## **3. The Core Template** 

Sub-form A (read-write, same direction): write = 0 for read in range(len(arr)): if arr[read] satisfies condition: arr[write] = arr[read] write += 1 Sub-form B (left-right, converging): left, right = 0, len(arr) - 1 while left < right: if some condition: swap(arr[left], arr[right]) left += 1 / right -= 1 else: move one or both pointers 

## **4. Two Representative Problems** 

PROBLEM A: Move Zeros to End Sheet: Lec 1 Easy 

**Recognition:** "Rearrange in-place, preserve relative order of non-zero elements, zeros at end." One category goes to the front (non-zeros), the other (zeros) fills remaining slots. Classic read-write. 

## **Approach:** 

- write pointer starts at 0. 

- read scans every element. 

- When arr[read] is non-zero, place it at arr[write] , advance write. 

- After the loop, fill positions from write to end with 0. 

## **Dry Run:** 

Array: [0, 1, 0, 3, 12] write = 0 

read=0, arr[0]=0:  skip read=1, arr[1]=1:  arr[write=0]=1, write=1  → [1, 1, 0, 3, 12] read=2, arr[2]=0:  skip read=3, arr[3]=3:  arr[write=1]=3, write=2  → [1, 3, 0, 3, 12] read=4, arr[4]=12: arr[write=2]=12, write=3 → [1, 3, 12, 3, 12] 

Fill from write=3 to end with 0: → [1, 3, 12, 0, 0] Answer: [1, 3, 12, 0, 0]  (correct) 

## **Code:** 

python def move_zeros(arr): write = 0 for read in range(len(arr)): if arr[read] != 0: arr[write] = arr[read] write += 1 while write < len(arr): arr[write] = 0 write += 1 

## **Complexity:** 

- Time: O(n) — one pass to compact, one pass to fill zeros. 

- Space: O(1) — no extra array. 

PROBLEM B: Sort 0s, 1s, and 2s (Dutch National Flag) Sheet: Lec 2 Medium 

**Recognition:** "Three categories, sort in-place, single pass." This is the canonical three-pointer problem. Do not sort — that is O(n log n). Do not count and overwrite — that is two passes. One pass with three pointers is the optimal. 

## **Approach:** 

- Three pointers: low , mid , high . 

- Invariant at all times: 

[0..low-1]  → all 0s  (confirmed zone) [low..mid-1]→ all 1s  (confirmed zone) [mid..high] → unknown (to be processed) 

- [high+1..n] → all 2s  (confirmed zone) 

   - mid is the scanner. Process arr[mid] : 

      - If 0: swap with arr[low] , advance both low and mid . 

      - If 1: it is in the right zone, just advance mid . 

      - If 2: swap with arr[high] , shrink high . Do NOT advance mid — what arrived at mid from high is unknown. 

## **Dry Run:** 

Array: [2, 0, 2, 1, 1, 0] low=0, mid=0, high=5 mid=0, arr[0]=2: swap(arr[0],arr[5]) → [0,0,2,1,1,2], high=4 mid=0, arr[0]=0: swap(arr[0],arr[0]) → [0,0,2,1,1,2], low=1,mid=1 mid=1, arr[1]=0: swap(arr[1],arr[1]) → [0,0,2,1,1,2], low=2,mid=2 mid=2, arr[2]=2: swap(arr[2],arr[4]) → [0,0,1,1,2,2], high=3 mid=2, arr[2]=1: mid=3 mid=3, arr[3]=1: mid=4 mid(4) > high(3) → STOP Result: [0, 0, 1, 1, 2, 2]  (correct) **Code:** python def sort_012(arr): low, mid, high = 0, 0, len(arr) - 1 

while mid <= high: if arr[mid] == 0: arr[low], arr[mid] = arr[mid], arr[low] low += 1 mid += 1 elif arr[mid] == 1: mid += 1 else: arr[mid], arr[high] = arr[high], arr[mid] high -= 1 # mid does NOT advance here 

## **Complexity:** 

- Time: O(n) — every element is processed at most once. 

- Space: O(1) — three integer pointers. 

## **5. Bonus: Left Rotate by D Places** 

This one appears simple but has a beautiful trick worth knowing cold. 

## **Approach (Reversal Algorithm):** 

To left-rotate by D: Step 1: Reverse arr[0..D-1] Step 2: Reverse arr[D..n-1] Step 3: Reverse entire arr[0..n-1] 

## **Why it works:** 

arr    = [1, 2, 3, 4, 5],  D=2 Step 1: reverse [1,2]     → [2, 1, 3, 4, 5] Step 2: reverse [3,4,5]   → [2, 1, 5, 4, 3] Step 3: reverse all       → [3, 4, 5, 1, 2] Correct. Three reversals, O(n) time, O(1) space. 

Each reversal is just the left-right converging sub-form of two pointers. 

## **6. Common Mistakes** 

Mistake 1: In Dutch National Flag, advancing mid after swapping with high. The swapped-in element is unprocessed — you must re-examine arr[mid] before moving forward. Mistake 2: In Move Zeros, trying to swap instead of overwrite. Swapping preserves zeros in the front half. Overwrite + fill is cleaner and equally O(n). 

Mistake 3: Forgetting D can be >= n. Always reduce D = D % n before rotating to avoid unnecessary work. Mistake 4: Left Rotate vs Right Rotate. Left by D is the same as Right by (n - D). Know which direction is asked. Mistake 5: In Remove Duplicates (Lec 1), returning the new length but forgetting to modify arr[0..new_length-1] in place. Interviewers check the array, not just the return value. 

## **7. How This Connects to Later Patterns** 

Two pointers is the workhorse of this entire sheet. 

→ P6 (kSum): After sorting, two pointers converge from both ends to find pairs summing to a target. Same left-right form. → P3 (Prefix Sum): Not directly two pointers, but the "read" and "write" mental model helps you see why prefix builds forward. → P4 (Sliding Window): Sliding window IS two pointers — a left and a right that both move rightward. The difference is the invariant: window maintains a condition, not a partition. 

→ Merge Two Sorted Arrays Without Extra Space (Lec 3 Hard): Uses two pointers on two separate arrays, then a reversal trick (gap method or two-pointer on the last n elements). Same toolkit, harder invariant. 

Pattern 2 problems to practice before moving on: Move Zeros to End                        [done above, warmup] Remove Duplicates from Sorted Array      [read-write, two ptr] Left Rotate by One / by D Places         [reversal trick] Sort 0s, 1s, 2s                          [done above, 3-pointer] Rearrange +/- Alternating                [tricky: order matters] Merge Two Sorted Arrays Without Space    [Lec 3 Hard — gap method] 

Ready for **Pattern 3: XOR Bit Tricks** ? 

## **You said: Continue but keep short** 

Continue but keep short 

May 29 

## **Claude responded: 1.** 

Prepared concise XOR bit tricks continuation Prepared concise XOR bit tricks continuation 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 3 — XOR BIT TRICKS                              ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

XOR has one property that makes it lethal for array problems: a ^ a = 0 and a ^ 0 = a . XOR every number with itself and it vanishes. XOR every number in a range, duplicates cancel, the lone survivor remains. 

## **2. The Two Weapons** 

Weapon 1: XOR all elements + XOR expected range → Duplicates cancel. What's left is the missing/extra number. 

Weapon 2: XOR all elements → All pairs cancel. The number appearing once survives. 

## **3. Two Representative Problems** 

PROBLEM A: Find Missing Number (1 to N) Sheet: Lec 1 Easy 

**Approach:** XOR all elements in arr, then XOR with 1 to N. Every number that appears in both cancels. The missing number never cancels. 

arr = [1, 3, 4, 5],  N=5, expected = {1,2,3,4,5} 

XOR of arr        = 1^3^4^5 XOR of 1..N       = 1^2^3^4^5 Total XOR         = (1^1)^(3^3)^(4^4)^(5^5)^2 = 0^0^0^0^2 = 2 Missing = 2  (correct) python def missing_number(arr, n): xor = 0 for i in range(1, n + 1): xor ^= i for x in arr: xor ^= x return xor 

Time: O(n) | Space: O(1) 

PROBLEM B: Number Appearing Once (others appear twice) Sheet: Lec 1 Easy 

**Approach:** XOR every element. All pairs cancel. Lone element survives. 

arr = [4, 1, 2, 1, 2] 

4^1^2^1^2 = 4^(1^1)^(2^2) = 4^0^0 = 4 

Answer: 4 python def single_number(arr): xor = 0 for x in arr: xor ^= x return xor Time: O(n) | Space: O(1) 

## **4. Common Mistakes** 

Mistake 1: Using sum formula instead of XOR for missing number — works but overflows for large N in languages without big integers. XOR never overflows. 

Mistake 2: Assuming XOR works when elements appear 3 times. It does not. That needs a different bit-counting approach. 

Mistake 3: For Repeating and Missing (Lec 3 Hard) — XOR gives you (missing ^ repeating). You then need a second step to separate them using the rightmost set bit. Do not skip that step. 

## **5. Connection to Other Patterns** 

XOR is a standalone tool. It plugs into P5 problems only. But the _thinking_ — "what cancels, what survives" — reappears in Boyer-Moore Voting (P8) where votes cancel instead of bits. 

Ready for **Pattern 4: Sliding Window** ? 

## **You said: Okay** 

Okay 

May 29 

## **Claude responded: 1.** 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 4 — SLIDING WINDOW                              ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

A caterpillar moves by stretching its front forward and pulling its tail up. It never jumps backward. That is a sliding window — a contiguous chunk of the array that grows from the right and shrinks from the left, always maintaining some condition. 

## **2. The Critical Constraint** 

Sliding window only works when values are non-negative. Why? Because adding an element always increases the sum. So if the window sum exceeds K, shrinking from the left will always help. With negatives, shrinking might make things worse — that is when you use Prefix Sum + HashMap (P3) instead. 

This distinction will save you in interviews. 

## **3. Core Template** 

left = 0 current_sum = 0 best = 0 for right in range(len(arr)): current_sum += arr[right]          # expand window while current_sum > K:             # condition violated current_sum -= arr[left]       # shrink window left += 1 

best = max(best, right - left + 1) # update answer 

## **4. Representative Problem** 

PROBLEM: Longest Subarray with Sum K (positives only) Sheet: Lec 1 Easy 

**Approach:** Expand right. If sum exceeds K, shrink from left until sum <= K. Track max window size when sum == K. 

arr = [1, 2, 3, 1, 1],  K = 3 left=0, sum=0, best=0 

right=0: sum=1, sum<K,  best=1  window=[1] right=1: sum=3, sum==K, best=2  window=[1,2] right=2: sum=6, sum>K shrink: sum=6-1=5, left=1 shrink: sum=5-2=3, left=2 sum==K, best=2          window=[3] right=3: sum=4, sum>K shrink: sum=4-3=1, left=3 best=2                  window=[1] right=4: sum=2, best=2          window=[1,1] 

Answer: 2  (subarrays [1,2] or [3]) python def longest_subarray_sum_k(arr, k): left = 0 current_sum = 0 best = 0 for right in range(len(arr)): current_sum += arr[right] 

while current_sum > k and left <= right: current_sum -= arr[left] left += 1 if current_sum == k: best = max(best, right - left + 1) return best 

Time: O(n) — each element enters and exits the window at most once. Space: O(1) 

## **5. Common Mistakes** 

Mistake 1: Using sliding window on arrays with negative numbers. → Use Prefix + HashMap (P3) instead. Mistake 2: Using while vs if to shrink. Always use while — a single shrink step may not be enough. Mistake 3: Checking best inside the while loop (shrinking phase) instead of after it. You update best only when window is valid. Mistake 4: Confusing "longest subarray with sum = K" (this problem) with "maximum subarray sum" (Kadane's, P9). Different goals, different tools. 

## **6. Connection to Other Patterns** 

Sliding window is two pointers (P2) with a sum-based invariant. When negatives enter → upgrade to Prefix + HashMap (P3). When you want max SUM not max LENGTH → upgrade to Kadane's (P9). These three form one family. Know when to switch. 

Ready for **Pattern 5: Prefix Sum + HashMap** ? 

## **You said: Okay** 

Okay 

May 29 

## **Claude responded: 1.** 

Charted algorithmic strategy using prefix sums and hash maps Charted algorithmic strategy using prefix sums and hash maps 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 5 — PREFIX SUM + HASHMAP                        ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

The sum of any subarray arr[i..j] equals prefix[j] - prefix[i-1] . So "find a subarray with sum K" becomes "find two prefix sums that differ by K." Store every prefix sum you've seen in a hashmap. For each new prefix sum, check if current - K already exists. If yes, you found your subarray. 

You are converting a subarray search into a hashmap lookup. 

## **2. Why Not Sliding Window Here** 

Negatives break sliding window because shrinking the window does not guarantee the sum decreases. Prefix + HashMap has no such restriction. It handles negatives, zeros, and positives equally. 

## **3. Core Template** 

prefix_sum = 0 map = {0: -1}        # sum 0 exists before index 0 for i in range(len(arr)): prefix_sum += arr[i] if (prefix_sum - K) in map: # subarray found from map[prefix_sum - K]+1 to i if prefix_sum not in map: map[prefix_sum] = i   # store earliest occurrence 

The {0: -1} seed is critical. It handles the case where the subarray starts from index 0. 

## **4. Two Representative Problems** 

PROBLEM A: Longest Subarray with Sum K (positives + negatives) Sheet: Lec 1 Easy 

**Approach:** Store earliest index of each prefix sum. When prefix - K exists in map, compute length and update best. 

arr = [1, -1, 5, -2, 3],  K = 3 map = {0: -1} 

i=0: prefix=1,  1-3=-2 not in map. map={0:-1, 1:0} i=1: prefix=0,  0-3=-3 not in map. map={0:-1, 1:0} (0 already in map, skip storing again — keep earliest) i=2: prefix=5,  5-3=2  not in map. map={0:-1,1:0,5:2} i=3: prefix=3,  3-3=0  in map at -1. length=3-(-1)=4. best=4 i=4: prefix=6,  6-3=3  in map? No.  map={..., 3:4, 6:4} Answer: 4  (subarray [1,-1,5,-2]) python def longest_subarray_sum_k(arr, k): prefix_sum = 0 best = 0 map = {0: -1} for i, x in enumerate(arr): prefix_sum += x if (prefix_sum - k) in map: best = max(best, i - map[prefix_sum - k]) if prefix_sum not in map: map[prefix_sum] = i return best 

Time: O(n) | Space: O(n) 

PROBLEM B: Count Subarrays with Given Sum K Sheet: Lec 2 Medium 

Same template, different goal. Now you COUNT occurrences, not track longest. So the map stores frequency, not index. And you DO add duplicates. 

arr = [1, 1, 1],  K = 2 map = {0: 1} i=0: prefix=1, 1-2=-1 not in map. map={0:1, 1:1} i=1: prefix=2, 2-2=0  in map, count += map[0] = 1. map={0:1,1:1,2:1} i=2: prefix=3, 3-2=1  in map, count += map[1] = 1. map={..., 3:1} Answer: 2  (subarrays [1,1] at indices 0-1 and 1-2) python def count_subarrays_sum_k(arr, k): prefix_sum = 0 count = 0 

map = {0: 1} for x in arr: prefix_sum += x count += map.get(prefix_sum - k, 0) map[prefix_sum] = map.get(prefix_sum, 0) + 1 return count 

Time: O(n) | Space: O(n) 

## **5. Common Mistakes** 

Mistake 1: Forgetting to seed map with {0: -1} or {0: 1}. → Misses subarrays starting from index 0. Single most common error on this pattern. 

Mistake 2: For LONGEST, storing the latest index instead of earliest. You want the longest subarray, so use the first time a prefix sum appeared. Mistake 3: For COUNT, storing index instead of frequency. The two sub-problems use the map differently — don't mix them. Mistake 4: Applying this to XOR problems without changing +/to ^ . The XOR subarray count problem (Lec 3) uses the same template with XOR replacing addition throughout. 

## **6. Connection to Other Patterns** 

This pattern directly solves two Lec 3 Hard problems: Largest Subarray with 0 Sum     → K=0, longest variant Count Subarrays with XOR = K    → replace + with ^, same map It is the upgrade path from Sliding Window (P4): positives only + length  → Sliding Window any values    + length   → Prefix + map (index) any values    + count    → Prefix + map (frequency) 

## Ready for **Pattern 6: Sort-based Multi-pointer (kSum)** ? 

## **You said: Okay** 

Okay 

May 29 

## **Claude responded: 1.** 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 6 — SORT-BASED MULTI-POINTER (kSUM)             ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

Imagine a number line. If you want two numbers that add to a target, and they are sorted, you place one finger at the left end and one at the right. If the sum is too small, move left finger right. Too large, move right finger left. You close in on the answer without checking every pair. 

Sorting costs O(n log n) once. After that, every additional pointer is O(n). This beats the brute force O(n^k) dramatically. 

## **2. The Generalization** 

2Sum → sort + 1 outer fix + 2 converging pointers  → O(n) 3Sum → sort + 1 outer fix + 2 converging pointers  → O(n²) 4Sum → sort + 2 outer fix + 2 converging pointers  → O(n³) kSum → sort + (k-2) outer loops + 2 converging     → O(n^(k-1)) 

The two converging pointers are always the innermost layer. Everything outside them is a fixed loop. 

## **3. Core Template (2Sum sorted version)** 

sort(arr) left, right = 0, len(arr) - 1 while left < right: s = arr[left] + arr[right] if s == target: record answer skip duplicates on both sides elif s < target: left += 1 else: right -= 1 

## **4. Two Representative Problems** 

PROBLEM A: 3-Sum Problem (find all unique triplets summing to 0) Sheet: Lec 3 Hard 

**Recognition:** "Triplets", "no duplicate triplets in output", "sum = 0." Fix one element, run 2Sum on the rest. 

## **Approach:** 

- Sort the array. 

- Fix i from 0 to n-3. Skip if arr[i] == arr[i-1] (duplicate). 

- Run two pointers left = i+1 , right = n-1 . 

- If sum == 0: record, skip duplicates on both pointers. 

- Skip duplicate values of i at the start of each outer iteration. 

arr = [-4,-1,-1,0,1,2]  (after sort) 

i=0, arr[i]=-4: left=1(-1), right=5(2) sum=-4+(-1)+2=-3 < 0 → left++ sum=-4+(-1)+2 ... left=2(-1), right=5(2) sum=-4+(-1)+2=-3 < 0 → left++ sum=-4+0+2=-2  < 0 → left++ sum=-4+1+2=-1  < 0 → left++ left=right, stop 

i=1, arr[i]=-1: left=2(-1), right=5(2) sum=-1+(-1)+2=0 → record [-1,-1,2] skip dup left → left=3, skip dup right → right=4 sum=-1+0+1=0  → record [-1,0,1] skip → left=4=right, stop 

i=2, arr[i]=-1: same as arr[1], SKIP (duplicate i) i=3, arr[i]=0: left=4(1), right=5(2) sum=0+1+2=3 > 0 → right-left=right, stop 

Answer: [[-1,-1,2], [-1,0,1]] python def three_sum(arr): arr.sort() result = [] n = len(arr) for i in range(n - 2): 

if i > 0 and arr[i] == arr[i - 1]: continue left, right = i + 1, n - 1 while left < right: s = arr[i] + arr[left] + arr[right] if s == 0: result.append([arr[i], arr[left], arr[right]]) while left < right and arr[left] == arr[left + 1]: left += 1 while left < right and arr[right] == arr[right - 1]: right -= 1 left += 1 right -= 1 elif s < 0: left += 1 else: right -= 1 return result 

Time: O(n²) | Space: O(1) excluding output 

PROBLEM B: 4-Sum Problem Sheet: Lec 3 Hard 

Same idea. Two outer fixed loops, one inner two-pointer sweep. Skip duplicates at every level. 

python def four_sum(arr, target): arr.sort() result = [] n = len(arr) for i in range(n - 3): if i > 0 and arr[i] == arr[i - 1]: continue for j in range(i + 1, n - 2): if j > i + 1 and arr[j] == arr[j - 1]: continue left, right = j + 1, n - 1 while left < right: s = arr[i] + arr[j] + arr[left] + arr[right] if s == target: result.append([arr[i],arr[j],arr[left],arr[right]]) while left < right and arr[left] == arr[left+1]: left += 1 

while left < right and arr[right] == arr[right-1]: right -= 1 left += 1 right -= 1 elif s < target: left += 1 else: right -= 1 return result 

Time: O(n³) | Space: O(1) excluding output 

## **5. Common Mistakes** 

Mistake 1: Skipping duplicate check for the outer fixed pointer. → You get duplicate triplets/quadruplets in output. The check is: if i > 0 and arr[i] == arr[i-1]: skip Not: if arr[i] == arr[i+1]: skip  (wrong direction) 

Mistake 2: After recording an answer in 3Sum, forgetting to move BOTH left and right. Moving only one causes an infinite loop or missed answers. 

Mistake 3: Skipping the inner duplicate check (on left/right) after recording. You will get repeated pairs in the output. Mistake 4: In 4Sum, overflow. arr[i]+arr[j]+arr[left]+arr[right] can exceed int range in Java/C++. Use long explicitly. 

Mistake 5: Using a HashSet to deduplicate output instead of in-place duplicate skipping. Works but is O(n) extra space and much slower in practice. Interviewers notice. 

## **6. Connection to Other Patterns** 

2Sum has two valid approaches: 

→ Sorted + two pointers (this pattern): O(n log n), O(1) space → HashMap (P5 variant): O(n) time, O(n) space Interviewer will ask about both. Know the tradeoff. 

3Sum and 4Sum are strictly this pattern — HashMap does not generalize cleanly to avoid duplicates across multiple fixed pointers. 

Find Union of Two Sorted Arrays (Lec 1) uses the same two-pointer merge idea: one pointer per array, advance the smaller one, skip duplicates. 

Ready for **Pattern 7: Boyer-Moore Voting** ? 

## **You said: Okay** 

Okay 

May 31 

## **Claude responded: 1.** 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 7 — BOYER-MOORE VOTING ALGORITHM                ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

Imagine a room full of people voting. Every time two people with different votes meet, they cancel each other out and both leave. If one candidate has a true majority, no matter how the cancellations happen, their supporters will always outnumber everyone else combined. At least one of theirs survives every cancellation. The last one standing is the majority candidate. 

That is Boyer-Moore. Cancellation instead of counting. 

## **2. Why Not a HashMap** 

HashMap gives you frequency counts in O(n) time but O(n) space. Boyer-Moore does it in O(1) space. For > n/2 majority that is strictly better. For > n/3 you need two candidates, still O(1). 

## **3. Core Template** 

> n/2 version (one candidate): 

candidate, count = None, 0 

for x in arr: if count == 0: candidate = x 

count += 1 if x == candidate else -1 

# candidate is the POTENTIAL majority. Verify with a second pass. 

## **4. Two Representative Problems** 

PROBLEM A: Majority Element > n/2 times Sheet: Lec 2 Medium 

## **Dry Run:** 

arr = [2, 2, 1, 1, 1, 2, 2] 

x=2: count=0 → candidate=2, count=1 x=2: count=1, x==candidate → count=2 x=1: count=2, x!=candidate → count=1 x=1: count=1, x!=candidate → count=0 x=1: count=0 → candidate=1, count=1 x=2: count=1, x!=candidate → count=0 x=2: count=0 → candidate=2, count=1 candidate = 2. Verify: 2 appears 4 times > 7/2. Correct. python def majority_element(arr): candidate, count = None, 0 for x in arr: if count == 0: candidate = x count += 1 if x == candidate else -1 # verify if arr.count(candidate) > len(arr) // 2: return candidate return -1 

Time: O(n) | Space: O(1) 

PROBLEM B: Majority Element > n/3 times Sheet: Lec 3 Hard 

At most 2 elements can appear more than n/3 times. Track two candidates simultaneously. Same cancellation logic — when an element matches neither candidate and both counts are non-zero, decrement both. 

arr = [1, 1, 1, 3, 3, 2, 2, 2] 

c1=None,cnt1=0 | c2=None,cnt2=0 x=1: cnt1=0 → c1=1, cnt1=1 x=1: x==c1  → cnt1=2 x=1: x==c1  → cnt1=3 x=3: cnt2=0 → c2=3, cnt2=1 x=3: x==c2  → cnt2=2 x=2: x!=c1,x!=c2,both>0 → cnt1=2, cnt2=1 x=2: x!=c1,x!=c2,both>0 → cnt1=1, cnt2=0 x=2: cnt2=0 → c2=2, cnt2=1 Candidates: 1 and 2. Verify: 1 appears 3 times > 8/3=2.67 ✓ 2 appears 3 times > 2.67 ✓ Answer: [1, 2] python def majority_element_n3(arr): c1, c2, cnt1, cnt2 = None, None, 0, 0 for x in arr: if x == c1: cnt1 += 1 elif x == c2: cnt2 += 1 elif cnt1 == 0: c1, cnt1 = x, 1 elif cnt2 == 0: c2, cnt2 = x, 1 else: cnt1 -= 1 cnt2 -= 1 # verify both candidates result = [] for c in [c1, c2]: if arr.count(c) > len(arr) // 3: result.append(c) return result 

Time: O(n) | Space: O(1) 

## **5. Common Mistakes** 

Mistake 1: Not verifying the candidate after the first pass. Boyer-Moore guarantees the majority element IS the candidate only if a majority element EXISTS. If none exists, the candidate is garbage. Always verify. 

Mistake 2: In n/3 version, checking x==c1 and x==c2 BEFORE checking if cnt==0. Order matters — if c1 is already set to x, you should increment, not replace c2 with x. 

Mistake 3: Assuming exactly 2 answers always exist for n/3. Zero or one majority element is also valid. Verify both. 

Mistake 4: Using > n/2 logic (one candidate) for the n/3 problem. You will miss the second majority element entirely. 

## **6. Connection to Other Patterns** 

Conceptually mirrors XOR (P3): both work by cancellation. XOR cancels bits. Voting cancels counts. 

The verification step is a single-pass track (P1) — just count occurrences of the candidate. These patterns layer. 

Ready for **Pattern 8: Kadane's Family** ? 

## **You said: Okay** 

Okay 

May 31 

## **Claude responded: 1.** 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 8 — KADANE'S FAMILY                             ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

At every index you face one decision: extend the current subarray, or throw it away and start fresh. You extend if the running sum is still positive — a negative running sum is a liability, it only 

drags down whatever comes next. You restart when carrying forward hurts more than starting clean. 

One decision per element. One pass. 

## **2. Why It Works** 

If the best subarray ends at index i , it either starts at i (fresh start) or extends from some earlier index where the prefix was positive. Kadane's captures exactly this by maintaining current — the best sum ending at the current position. 

## **3. Core Template** 

current = arr[0] best    = arr[0] for i in range(1, len(arr)): current = max(arr[i], current + arr[i]) best    = max(best, current) return best 

Do not initialize current = 0 . That silently assumes an empty subarray is valid, which breaks on all-negative arrays. 

## **4. Two Representative Problems** 

PROBLEM A: Kadane's Algorithm — Maximum Subarray Sum Sheet: Lec 2 Medium arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4] 

i=1: x=1,  current=max(1, -2+1)=max(1,-1)=1,  best=1 i=2: x=-3, current=max(-3,1-3)=max(-3,-2)=-2, best=1 i=3: x=4,  current=max(4,-2+4)=max(4,2)=4,    best=4 i=4: x=-1, current=max(-1,4-1)=max(-1,3)=3,   best=4 i=5: x=2,  current=max(2,3+2)=5,               best=5 i=6: x=1,  current=max(1,5+1)=6,               best=6 i=7: x=-5, current=max(-5,6-5)=1,              best=6 i=8: x=4,  current=max(4,1+4)=5,               best=6 Answer: 6  (subarray [4,-1,2,1]) python def max_subarray(arr): current = best = arr[0] 

for x in arr[1:]: current = max(x, current + x) best = max(best, current) return best 

Time: O(n) | Space: O(1) 

PROBLEM B: Maximum Product Subarray Sheet: Lec 3 Hard 

**Why it differs from sum:** A negative times a negative becomes positive. So you cannot discard the minimum — it might become the maximum after the next negative. Track both max_prod and min_prod at every step. 

arr = [2, 3, -2, 4] 

curr_max=2, curr_min=2, best=2 x=3:  new_max=max(3, 2*3, 2*3)=6 new_min=min(3, 2*3, 2*3)=3,  best=6 x=-2: new_max=max(-2, 6*-2, 3*-2)=max(-2,-12,-6)=-2 new_min=min(-2, 6*-2, 3*-2)=min(-2,-12,-6)=-12, best=6 x=4:  new_max=max(4, -2*4, -12*4)=max(4,-8,-48)=4 new_min=min(4, -2*4, -12*4)=min(4,-8,-48)=-48, best=6 Answer: 6  (subarray [2,3]) python def max_product(arr): curr_max = curr_min = best = arr[0] for x in arr[1:]: candidates = (x, curr_max * x, curr_min * x) curr_max, curr_min = max(candidates), min(candidates) best = max(best, curr_max) return best 

Time: O(n) | Space: O(1) 

## **5. Common Mistakes** 

Mistake 1: Initializing current = 0. Breaks on all-negative 

arrays — returns 0 instead of the least negative element. 

Mistake 2: In product variant, updating curr_max before computing new curr_min using the old curr_max. Compute both from the SAME previous values. Use a temp or tuple. 

Mistake 3: Confusing this with "longest subarray with sum K" (P4/P5). Kadane finds maximum VALUE. P4/P5 find maximum LENGTH for a fixed target. Different goals entirely. 

Mistake 4: Not handling the "print the subarray" variant — track start, end, and temp_start indices separately. Reset temp_start when you restart fresh. 

## **6. Connection to Other Patterns** 

Kadane is single-pass tracking (P1) with a two-variable state. Product variant adds one more tracked variable. 

The "print subarray" extension (Lec 2) is pure bookkeeping on top — same logic, extra index variables. 

Kadane recurs in DP later: it is the 1D case of maximum submatrix sum problems. Once you own this, those problems reduce to "run Kadane on each row compression." 

## Ready for **Pattern 9: Greedy Observation** ? 

## **You said: Okay** 

Okay 

May 31 

## **Claude responded: 1.** 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 9 — GREEDY OBSERVATION                          ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

Greedy means: make the best local choice at each step and never revisit it. It works only when the problem has an _exchange argument_ — you can prove that swapping any locally suboptimal choice for the greedy one never makes the global answer worse. 

Unlike DP, you do not store subproblem results. Unlike brute force, you do not explore all options. You just act on what looks best right now. 

## **2. Why This Is Hard** 

The difficulty is not implementation — greedy code is usually short. The difficulty is _recognizing_ that greedy works and _knowing what the greedy choice is_ . These two problems from the sheet each require a non-obvious observation before the code becomes trivial. 

## **3. Two Representative Problems** 

PROBLEM A: Next Permutation Sheet: Lec 2 Medium 

**Recognition:** "Find the next lexicographically greater arrangement." No brute force over all permutations. One linear scan with a precise set of steps. 

**Observation:** The array is as large as possible from some index rightward (it is non-increasing from the right). To get the next permutation, find the leftmost place where this breaks, make the smallest possible increase there, and make everything to its right as small as possible. 

## **Three-step algorithm:** 

Step 1: Find the rightmost index i where arr[i] < arr[i+1]. (the "dip" — where the sequence stops being non-increasing) Step 2: Find the rightmost index j where arr[j] > arr[i]. Swap arr[i] and arr[j]. Step 3: Reverse arr[i+1 .. end]. (makes the suffix as small as possible) Edge case: if no dip exists, the array is fully non-increasing — it is the LAST permutation. Reverse the entire array. 

## **Dry Run:** 

arr = [1, 3, 5, 4, 2] 

Step 1: scan right to left for dip i=3: arr[3]=4 < arr[4]=2? No i=2: arr[2]=5 < arr[3]=4? No i=1: arr[1]=3 < arr[2]=5? YES → dip at i=1 

Step 2: rightmost j where arr[j] > arr[1]=3 j=3: arr[3]=4 > 3? YES → j=3 swap arr[1] and arr[3] → [1, 4, 5, 3, 2] Step 3: reverse arr[2..4] → [1, 4, 2, 3, 5] Answer: [1, 4, 2, 3, 5]  (correct) python def next_permutation(arr): n = len(arr) i = n - 2 while i >= 0 and arr[i] >= arr[i + 1]: i -= 1 if i >= 0: j = n - 1 while arr[j] <= arr[i]: j -= 1 arr[i], arr[j] = arr[j], arr[i] arr[i + 1:] = reversed(arr[i + 1:]) 

Time: O(n) | Space: O(1) 

PROBLEM B: Merge Overlapping Subintervals Sheet: Lec 3 Hard 

**Recognition:** "Given intervals, merge all overlapping ones." Sort by start. Then one greedy pass: if the current interval overlaps with the last merged one, extend it. Otherwise start a new merged interval. 

**Greedy choice:** Always extend the current merged interval as far right as possible before committing. 

## **Dry Run:** 

intervals = [[1,3],[2,6],[8,10],[15,18]] Sort by start → already sorted result = [[1,3]] [2,6]: start=2 <= last_end=3 → overlap. extend end to max(3,6)=6 result = [[1,6]] [8,10]: start=8 > last_end=6 → no overlap. append. 

result = [[1,6],[8,10]] [15,18]: start=15 > last_end=10 → no overlap. append. result = [[1,6],[8,10],[15,18]] Answer: [[1,6],[8,10],[15,18]] python def merge_intervals(intervals): intervals.sort(key=lambda x: x[0]) result = [intervals[0]] for start, end in intervals[1:]: if start <= result[-1][1]: result[-1][1] = max(result[-1][1], end) else: result.append([start, end]) return result 

Time: O(n log n) — dominated by sort | Space: O(n) for output 

## **4. Common Mistakes** 

Mistake 1: Next Permutation — scanning left to right for the dip instead of right to left. You need the RIGHTMOST dip to make the smallest possible change. 

Mistake 2: Next Permutation — after the swap, sorting the suffix instead of reversing it. Since the suffix is already non-increasing after the swap, reversing is O(n). Sorting is O(n log n) and unnecessary. 

Mistake 3: Merge Intervals — forgetting to sort first. Without sorting, you cannot make the greedy comparison linearly. Mistake 4: Merge Intervals — using result[-1][1] = end instead of result[-1][1] = max(result[-1][1], end). One interval can be fully contained inside another: [[1,10],[2,5]]. Without max, you would shrink the merged interval to 5. 

## **5. Connection to Other Patterns** 

Next Permutation is self-contained but its suffix reversal step is Two Pointers (P2) in disguise. 

Merge Intervals reappears heavily in interval scheduling problems across Greedy (Step 12 of the sheet) and in calendar/meeting room problems at FAANG. The pattern is identical — sort by start, process linearly. The greedy mindset here ("sort, then one linear pass") is the same structure you will use for Jump Game, N Meetings in a Room, and Job Sequencing in Step 12. 

## Ready for **Pattern 10: Divide & Conquer (Merge Sort Applications)** ? 

## **You said: Okay** 

Okay 

May 31 

## **Claude responded: 1.** 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 10 — DIVIDE & CONQUER (MERGE SORT APPLICATIONS) ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

Merge sort does something most people ignore: while merging two sorted halves, it knows something powerful — every element in the left half comes before every element in the right half by index. This means during the merge step you can count cross-half pairs satisfying some condition _for free_ , without a nested loop. 

The array is split until trivial, then rebuilt. Counting happens during the rebuild. 

## **2. Why Not Brute Force** 

Both problems here ask "count pairs (i,j) where i < j and some condition." Brute force checks all pairs: O(n²). Merge sort extracts the count during the merge phase: O(n log n). The index ordering guarantee is what makes it work — you cannot get this from a hash map. 

## **3. Core Template** 

def merge_sort(arr, left, right): if left >= right: 

return 0          # base case, no pairs in one element mid = (left + right) // 2 count  = merge_sort(arr, left, mid) count += merge_sort(arr, mid+1, right) count += merge(arr, left, mid, right)   # count cross pairs HERE return count 

Everything changes per problem is inside merge() . 

## **4. Two Representative Problems** 

PROBLEM A: Count Inversions Sheet: Lec 3 Hard 

An inversion is a pair (i,j) where i < j but arr[i] > arr[j]. 

**Observation during merge:** Left half is sorted, right half is sorted. When you pick an element from the right half over one from the left half (because right < left), every remaining element in the left half also forms an inversion with this right element. Count them all at once. 

## **Dry Run:** 

arr = [5, 3, 2, 4, 1] 

Split → [5,3,2] and [4,1] Split → [5,3] and [2]    |  [4] and [1] Merge [5,3]: right(3) < left(5) → inversions += 1 (one element left in left half) merged = [3,5], count=1 Merge [3,5] and [2]: right(2) < left(3) → inversions += 2 (two elements remain: 3,5) merged = [2,3,5], count=3 Merge [4] and [1]: right(1) < left(4) → inversions += 1 merged = [1,4], count=4 Merge [2,3,5] and [1,4]: right(1) < left(2) → inversions += 3 (three elements: 2,3,5) right(4) > left(2) → take 2, no count right(4) > left(3) → take 3, no count right(4) < left(5) → inversions += 1 

merged = [1,2,3,4,5], count=8 

Answer: 8 python def merge_count(arr, left, mid, right): l, r = left, mid + 1 temp = [] count = 0 while l <= mid and r <= right: if arr[l] <= arr[r]: temp.append(arr[l]); l += 1 else: count += (mid - l + 1) # all remaining left elements temp.append(arr[r]); r += 1 while l <= mid: temp.append(arr[l]); l += 1 while r <= right: temp.append(arr[r]); r += 1 arr[left:right+1] = temp return count def count_inversions(arr, left, right): if left >= right: return 0 mid = (left + right) // 2 count  = count_inversions(arr, left, mid) count += count_inversions(arr, mid+1, right) count += merge_count(arr, left, mid, right) return count 

Time: O(n log n) | Space: O(n) for temp array 

PROBLEM B: Reverse Pairs Sheet: Lec 3 Hard 

Count pairs (i,j) where i < j and arr[i] > 2 * arr[j]. 

**Key difference from inversions:** The condition is arr[i] > 2*arr[j] , not arr[i] > arr[j] . You CANNOT count during the merge step directly because swapping elements to merge would invalidate the count condition. Instead: count BEFORE merging, then merge separately. 

python def count_pairs(arr, left, mid, right): 

count = 0 j = mid + 1 for i in range(left, mid + 1): while j <= right and arr[i] > 2 * arr[j]: j += 1 count += (j - mid - 1) return count def merge(arr, left, mid, right): l, r = left, mid + 1 temp = [] while l <= mid and r <= right: if arr[l] <= arr[r]: temp.append(arr[l]); l += 1 else: temp.append(arr[r]); r += 1 while l <= mid: temp.append(arr[l]); l += 1 while r <= right: temp.append(arr[r]); r += 1 arr[left:right+1] = temp 

def reverse_pairs(arr, left, right): if left >= right: return 0 mid = (left + right) // 2 count  = reverse_pairs(arr, left, mid) count += reverse_pairs(arr, mid+1, right) count += count_pairs(arr, left, mid, right) # count FIRST merge(arr, left, mid, right) # then merge return count 

Time: O(n log n) | Space: O(n) 

## **5. Common Mistakes** 

Mistake 1: In Reverse Pairs, counting DURING merge instead of BEFORE. The 2*arr[j] condition does not hold after elements are rearranged. Always count first, then merge. 

Mistake 2: In Count Inversions, forgetting the bulk count. When arr[r] < arr[l], ALL elements from l to mid are inverted with arr[r]. Count is (mid - l + 1), not 1. 

Mistake 3: Not copying temp back into arr. The sort must actually happen for future merge steps to work correctly. Mistake 4: Confusing the two problems. Inversions: count DURING merge. Reverse Pairs: count BEFORE merge. This distinction is the entire interview point of these problems. 

## **6. Connection to Other Patterns** 

This is the only pattern on the sheet that uses recursion as the core mechanism. You will see this mindset again in: → Binary Trees (traversal + counting during merge of subtrees) → Segment Trees (range queries built on divide & conquer) The merge step itself is Two Pointers (P2) — two pointers on two sorted halves moving forward. Patterns compose. These two problems are Lec 3 Hard for a reason. If you can implement both cleanly from scratch in an interview, you signal mastery of recursion, sorting, and counting together. 

Ready for **Pattern 11: Matrix Traversal & Manipulation** ? 

## **You said: Okay** 

Okay 

May 31 

## **Claude responded: 1.** 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 11 — MATRIX TRAVERSAL & MANIPULATION            ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

A 2D array is just a 1D array with two indices. Most matrix problems are not algorithmically deep — they test your ability to simulate a process precisely without losing track of boundaries, directions, or index arithmetic. The difficulty is implementation discipline, not insight. 

Three distinct sub-problems on the sheet. Each has one key trick. 

## **2. Three Problems** 

PROBLEM A: Set Matrix Zeros Sheet: Lec 2 Medium If arr[i][j] == 0, set entire row i and column j to 0. 

**The trap:** If you zero out rows and columns as you find zeros, new zeros contaminate future iterations. You cannot tell original zeros from propagated ones. 

**Key trick:** First pass — mark which rows and columns need zeroing. Second pass — apply the zeros. 

Naive marking uses O(m+n) extra space (a row array and column array). Optimal uses the first row and first column of the matrix itself as the markers, plus two flags for whether row-0 and col-0 themselves need zeroing. 

python def set_zeros(matrix): m, n = len(matrix), len(matrix[0]) first_row_zero = any(matrix[0][j] == 0 for j in range(n)) first_col_zero = any(matrix[i][0] == 0 for i in range(m)) # use first row/col as markers for rest of matrix for i in range(1, m): for j in range(1, n): if matrix[i][j] == 0: matrix[i][0] = 0 matrix[0][j] = 0 # zero out cells based on markers for i in range(1, m): for j in range(1, n): if matrix[i][0] == 0 or matrix[0][j] == 0: matrix[i][j] = 0 if first_row_zero: for j in range(n): matrix[0][j] = 0 if first_col_zero: for i in range(m): matrix[i][0] = 0 

Time: O(m*n) | Space: O(1) 

PROBLEM B: Rotate Image by 90 Degrees (clockwise) Sheet: Lec 2 Medium 

**Key trick:** Clockwise 90° rotation = Transpose + Reverse each row. 

Why? After transpose, arr[i][j] goes to arr[j][i] . After reversing each row, arr[j][i] goes to arr[j][n-1-i] . That is exactly the clockwise rotation formula (i,j) `→` (j, n-1-i) . 

Original:        Transpose:       Reverse rows: 1  2  3          1  4  7          7  4  1 4  5  6    →     2  5  8    →     8  5  2 7  8  9          3  6  9          9  6  3 python def rotate(matrix): n = len(matrix) # transpose for i in range(n): for j in range(i + 1, n): matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j] # reverse each row for row in matrix: row.reverse() 

Time: O(n²) | Space: O(1) 

Counter-clockwise = Transpose + Reverse each column (or reverse rows first, then transpose). 

PROBLEM C: Spiral Traversal of Matrix Sheet: Lec 2 Medium 

**Key trick:** Maintain four boundaries — top , bottom , left , right . Peel one layer at a time. After traversing each side, shrink the corresponding boundary inward. 

Matrix:          Spiral order: 1  2  3 4  5  6    →    [1,2,3,6,9,8,7,4,5] 7  8  9 Layer 1: top row    (left→right): 1,2,3   → top++ 

right col  (top→bottom): 6,9     → right-bottom row (right→left): 8,7     → bottom-left col   (bottom→top): 4       → left++ Layer 2: top row: 5  → top++ top > bottom now, stop python def spiral_order(matrix): result = [] top, bottom = 0, len(matrix) - 1 left, right = 0, len(matrix[0]) - 1 while top <= bottom and left <= right: for j in range(left, right + 1): result.append(matrix[top][j]) top += 1 for i in range(top, bottom + 1): result.append(matrix[i][right]) right -= 1 if top <= bottom: for j in range(right, left - 1, -1): result.append(matrix[bottom][j]) bottom -= 1 if left <= right: for i in range(bottom, top - 1, -1): result.append(matrix[i][left]) left += 1 return result 

Time: O(m*n) | Space: O(1) excluding output 

## **3. Common Mistakes** 

Mistake 1: Set Matrix Zeros — zeroing in-place during the first pass. You destroy the information you need for later cells. Always decouple the "mark" pass from the "zero" pass. Mistake 2: Rotate — trying to do it in one pass with index math instead of the two-step trick. Transpose + reverse is cleaner, faster to implement under pressure, and harder to get wrong. 

Mistake 3: Spiral — not guarding the bottom row and left column 

traversals with boundary checks (top <= bottom and left <= right). Single-row or single-column matrices will double-count without them. Mistake 4: Spiral — decrementing the boundary AFTER the loop instead of immediately after. Boundary must shrink before the next side starts or you re-traverse cells. 

## **4. Connection to Other Patterns** 

These three are largely self-contained. But: 

→ Set Matrix Zeros uses the "mark first, apply later" idea that reappears in graph problems (BFS/DFS marking visited nodes before processing). → Spiral and Rotate index arithmetic reappears in diagonal traversals and search in sorted matrices (Step 4 Binary Search on 2D arrays). Matrix problems at FAANG are rarely just these three — they test whether you can adapt the same boundary/index discipline to novel traversal patterns. Own the template, not the memorized solution. 

Ready for **Pattern 12: Mathematical Construction** — the final pattern? 

## **You said: Okay** 

Okay 

May 31 

## **Claude responded: 1.** 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║         PATTERN 12 — MATHEMATICAL CONSTRUCTION                  ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

## **1. Core Intuition** 

Some problems have a mathematical structure so clean that the algorithm falls directly out of the formula. You do not search, sort, or slide. You construct the answer by knowing a rule and applying it precisely. The insight is purely mathematical — once you see it, the code is trivial. 

## **2. Two Problems** 

PROBLEM A: Longest Consecutive Sequence Sheet: Lec 2 Medium (also listed under P12 here) 

**Recognition:** "Find longest sequence of consecutive integers, O(n) time." Sorting gives O(n log n). The O(n) solution uses a hash set with one key observation. 

**Key insight:** Only start counting from the beginning of a sequence. An element x is the start of a sequence only if x-1 is NOT in the set. If x-1 exists, x is in the middle — skip it. This ensures each element is visited at most twice total across all sequences. 

arr = [100, 4, 200, 1, 3, 2] set = {100, 4, 200, 1, 3, 2} x=100: 99 not in set → start. streak: 100,101? No. length=1 x=4:   3  in set → skip x=200: 199 not in set → start. streak: 200,201? No. length=1 x=1:   0  not in set → start. streak: 1,2,3,4,5? 1✓ 2✓ 3✓ 4✓ 5✗ → length=4 x=3:   2  in set → skip x=2:   1  in set → skip Answer: 4  (sequence 1,2,3,4) python def longest_consecutive(arr): num_set = set(arr) best = 0 for x in num_set: if x - 1 not in num_set: # start of sequence length = 1 while x + length in num_set: length += 1 best = max(best, length) return best 

Time: O(n) — each element enters the while loop at most once total Space: O(n) for the set 

PROBLEM B: Pascal's Triangle Sheet: Lec 3 Hard 

Three variants asked on the sheet: 

1. Print entire triangle up to N rows 

2. Print Nth row only 

3. Find element at (r, c) 

**Key insight:** Pascal's triangle is binomial coefficients. Element at row r , column c (1-indexed) is C(r-1, c-1). Each row is built from the previous: row[j] = row[j-1] * (r - j) / j . 

**Variant 3 — single element at (r,c):** Direct formula C(r-1,c-1). 

python def pascal_element(r, c): # C(r-1, c-1) result = 1 for i in range(c - 1): result = result * (r - 1 - i) // (i + 1) return result 

**Variant 2 — print Nth row:** Use the multiplicative formula, build left to right. 

Row 5: start with 1 1 * (5-1)/1 = 4       → [1, 4] 4 * (5-2)/2 = 6       → [1, 4, 6] 6 * (5-3)/3 = 4       → [1, 4, 6, 4] 4 * (5-4)/4 = 1       → [1, 4, 6, 4, 1] python def nth_row(n): row = [1] val = 1 for j in range(1, n): val = val * (n - j) // j row.append(val) return row 

**Variant 1 — entire triangle:** Call nth_row for each row, or build iteratively where each row derives from the previous. 

python def pascal_triangle(n): triangle = [] for i in range(1, n + 1): 

triangle.append(nth_row(i)) return triangle 

Time: O(n²) for full triangle | Space: O(n) per row 

## **3. Common Mistakes** 

Mistake 1: Consecutive Sequence — iterating over arr instead of num_set in the outer loop. Duplicates in arr cause redundant work and wrong streak counts. 

Mistake 2: Consecutive Sequence — not checking x-1 before starting a streak. Without this guard every element triggers a streak and you get O(n²) in the worst case. 

Mistake 3: Pascal's Triangle — using integer division too early in the multiplicative formula. Always multiply before dividing: val = val * (n-j) // j, not val = val * ((n-j) // j). The latter truncates prematurely for odd numerators. 

Mistake 4: Pascal's Triangle — building it with the additive method (triangle[i][j] = triangle[i-1][j-1] + triangle[i-1][j]) for a single element query. That is O(n²). Use the direct formula for single element — O(r) time. 

## **4. Connection to Other Patterns** 

Consecutive Sequence uses a hash set the same way P3 uses a hash map — O(1) lookup replaces O(n) scanning. 

Pascal's Triangle is pure combinatorics. The multiplicative row formula reappears in probability problems and combinatorics questions at FAANG. Know it cold. 

These two are the "math knowledge" checkpoint on the sheet. If you reach them and they feel easy, your fundamentals across all 12 patterns are solid. 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║                  ALL 12 PATTERNS — COMPLETE                     ║ ╠═══════════════════════════════════════════════════════════ ═══════╣ 

**==> picture [468 x 302] intentionally omitted <==**

**----- Start of picture text -----**<br>
║  P1  Single-pass Tracking         P7  Greedy Observation        ║<br>║  P2  Two Pointers / In-place      P8  Boyer-Moore Voting        ║<br>║  P3  XOR Bit Tricks               P9  Kadane's Family           ║<br>║  P4  Sliding Window               P10 Divide & Conquer          ║<br>║  P5  Prefix Sum + HashMap         P11 Matrix Traversal          ║<br>║  P6  Sort-based Multi-pointer     P12 Mathematical Construction ║<br>╠═══════════════════════════════════════════════════════════<br>═══════╣<br>║  MASTER CHEAT                                                    ║<br>║  Subarray + max sum?       → Kadane (P9)                        ║<br>║  Subarray + sum=K + neg?   → Prefix+Map (P5)                    ║<br>║  Subarray + sum=K + pos?   → Sliding Window (P4)                ║<br>║  Pairs with target sum?    → Sort+TwoPtr (P6) or Map (P5)       ║<br>║  Count pairs (i<j)?        → Merge Sort (P10)                   ║<br>║  Missing/once element?     → XOR (P3)                           ║<br>║  >n/k majority?            → Voting (P8)                        ║<br>║  In-place rearrange?       → Two Pointers (P2)                  ║<br>║  Matrix transform?         → P11                                ║<br>║  Generate/find formula?    → P12                                ║<br>╚═══════════════════════════════════════════════════════════<br>═══════╝<br>**----- End of picture text -----**<br>


That completes the full Arrays pattern map and all 12 teaching sessions. You now have the complete mental model for every problem in Step 3 of Striver's A2Z sheet. 

What would you like to do next — practice problems, move to Step 4 (Binary Search), or revisit any pattern? 

## **You said: Give me an small mental map for quick recovery of what's done and an map for quick recap ( very short one )** 

Give me an small mental map for quick recovery of what's done and an map for quick recap ( very short one ) 

May 31 

## **Claude responded: ---** 

╔═══════════════════════════════════════════════════════════ ═══════╗ 

║                    WHAT WE COVERED                               ║ 

╚═══════════════════════════════════════════════════════════ ═══════╝ 

STRIVER A2Z — STEP 3: ARRAYS (40 problems) Lec1 Easy (14) → Lec2 Medium (14) → Lec3 Hard (12) 12 Patterns derived. All 40 problems mapped. 

P1  Single-pass Tracking      [Lec1+2]  ✓ P2  Two Pointers / In-place   [Lec1+3]  ✓ P3  XOR Bit Tricks            [Lec1+3]  ✓ P4  Sliding Window            [Lec1]    ✓ P5  Prefix Sum + HashMap      [Lec1+2+3]✓ P6  Sort-based kSum           [Lec1+3]  ✓ P7  Greedy Observation        [Lec2+3]  ✓ P8  Boyer-Moore Voting        [Lec2+3]  ✓ P9  Kadane's Family           [Lec2+3]  ✓ P10 Divide & Conquer          [Lec3]    ✓ P11 Matrix Traversal          [Lec2]    ✓ P12 Mathematical Construction [Lec2+3]  ✓ 

╔═══════════════════════════════════════════════════════════ ═══════╗ ║                     QUICK RECAP MAP                              ║ ╚═══════════════════════════════════════════════════════════ ═══════╝ 

P1  SINGLE-PASS      one loop, one state var tools: max/min/streak variable probs: largest, 2nd largest, stock, leaders 

P2  TWO POINTERS     read+write OR left+right tools: two cursors, no extra array 

probs: move zeros, sort012, rotate, remove dups 

P3  XOR              a^a=0, lone survives tools: XOR all + XOR range probs: missing number, single number 

P4  SLIDING WINDOW   expand right, shrink left tools: two ptrs + running sum probs: longest subarray sumK (positives only) rule:  ONLY for non-negative arrays 

P5  PREFIX+MAP       sum[0..j]-sum[0..i]=subarray 

tools: hashmap of prefix sums probs: longest/count subarray sumK, XOR subarrays rule:  use when negatives present or count needed 

P6  kSUM             sort once, fix+converge tools: sort + nested loops + two ptrs probs: 2sum, 3sum, 4sum, union 

P7  GREEDY           local best = global best tools: sort + one linear pass probs: next permutation, merge intervals 

P8  VOTING           cancellation, survivor wins tools: candidate + count var probs: majority >n/2, majority >n/3 rule:  ALWAYS verify candidate after first pass 

P9  KADANE           extend or restart tools: curr and best variables probs: max subarray sum, max product subarray rule:  init to arr[0] not 0 

P10 MERGE SORT       count during/before merge tools: divide + merge + count cross pairs probs: count inversions, reverse pairs rule:  inversions=count DURING, revpairs=BEFORE P11 MATRIX           boundaries + index arithmetic tools: top/bottom/left/right walls probs: set zeros, rotate 90°, spiral 

P12 MATH             formula over brute force tools: hash set, binomial coefficient probs: longest consecutive, pascal's triangle 

══════════════════════════════════════════════════ WHEN STUCK — ASK THESE IN ORDER: 

1. Subarray?      → P4/P5/P9 2. Pairs/count?   → P5/P6/P10 3. In-place?      → P2 4. Missing/lone?  → P3 5. Majority?      → P8 6. Matrix?        → P11 7. Generate?      → P12 

══════════════════════════════════════════════════ 

## **You said: How to solve this then : https://leetcode.** 

How to solve this then : https://leetcode.com/problems/set-matrix-zeroes/ 

## 3:57 PM 

## **Claude responded: No worries, this is LeetCode 73 — Set Matrix Zeroes.** 

Failed to fetch: https://leetcode.com/problems/set-matrix-zeroes/ Failed to fetch: https://leetcode.com/problems/set-matrix-zeroes/ 

No worries, this is LeetCode 73 — Set Matrix Zeroes. We covered this fully under P11. Here is the direct solution walkthrough. 

**Pattern:** P11 Matrix Traversal. **Technique:** Mark first, apply later. 

## **The three approaches — know all three, interviewers ask about tradeoffs.** 

Approach 1: O(m*n) space  — copy entire matrix. Naive. Approach 2: O(m+n) space  — separate row[] and col[] boolean arrays. Approach 3: O(1) space    — use first row and col as markers. Optimal. 

## **Approach 2 — O(m+n), easiest to explain first** 

python def setZeroes(matrix): m, n = len(matrix), len(matrix[0]) rows = [False] * m cols = [False] * n for i in range(m): for j in range(n): if matrix[i][j] == 0: rows[i] = True cols[j] = True 

for i in range(m): 

for j in range(n): if rows[i] or cols[j]: matrix[i][j] = 0 

## **Approach 3 — O(1) space, what the interviewer wants** 

Use matrix[0][j] to mark which columns need zeroing. Use matrix[i][0] to mark which rows need zeroing. But first row and first column are themselves data — save their zero-status in two flags before overwriting them. 

Step 1: Check if first row has any zero  → flag first_row Step 2: Check if first col has any zero  → flag first_col Step 3: Scan inner matrix (i>=1, j>=1). If zero found: mark matrix[i][0]=0 and matrix[0][j]=0 Step 4: Use those marks to zero inner matrix Step 5: Apply first_row and first_col flags last python def setZeroes(matrix): m, n = len(matrix), len(matrix[0]) first_row = any(matrix[0][j] == 0 for j in range(n)) first_col = any(matrix[i][0] == 0 for i in range(m)) # mark using first row and col as signal arrays for i in range(1, m): for j in range(1, n): if matrix[i][j] == 0: matrix[i][0] = 0 matrix[0][j] = 0 # zero inner matrix based on marks for i in range(1, m): for j in range(1, n): if matrix[i][0] == 0 or matrix[0][j] == 0: matrix[i][j] = 0 # handle first row and col separately if first_row: for j in range(n): matrix[0][j] = 0 if first_col: for i in range(m): matrix[i][0] = 0 

## **Dry run on LC example:** 

Input: 1  1  1 1  0  1 1  1  1 

first_row = False (no zero in row 0) first_col = False (no zero in col 0) Scan inner (i>=1, j>=1): i=1,j=1: matrix[1][1]=0 → mark matrix[1][0]=0, matrix[0][1]=0 

Matrix after marking: 1  0  1 0  0  1 1  1  1 

Zero inner based on marks: i=1: matrix[1][0]=0 → entire row 1 zeros j=1: matrix[0][1]=0 → entire col 1 zeros 

Result: 1  0  1 0  0  0 1  0  1 

first_row=False, first_col=False → skip 

Output: 1  0  1 0  0  0 1  0  1  ✓ 

## **The one mistake everyone makes:** 

Applying first_row and first_col BEFORE the inner matrix step. If first_row is true and you zero row 0 early, those zeros corrupt your column markers in matrix[0][j] . Always handle them last. 

Time: O(m*n) | Space: O(1) 

