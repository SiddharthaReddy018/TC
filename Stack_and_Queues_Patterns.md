# Stack and Queues — Pattern-wise Notes (Striver A2Z)

---

## PATTERN 1 — DS IMPLEMENTATION

### 1. Core Intuition
A stack is a pile of plates — you only ever touch the top one (LIFO). A queue is a line at a ticket counter — you join at the back, you're served from the front (FIFO). Everything else in this pattern is just: "given pile-of-plates rules, build the pile using array/linked-list parts" or "given ticket-line rules, build the line using pile-of-plates parts".

### 2. Why This Pattern Exists
Interviews test whether you actually understand the LIFO/FIFO *contracts*, not just whether you can call `.push()`. Forcing you to build a queue out of stacks (or vice versa) exposes whether you understand that the only thing that matters is the *order elements come out*, regardless of what's underneath.

### 3. Core Template / Technique
- Stack via array: maintain `top` index. push -> `arr[++top]=x`. pop -> `arr[top--]`.
- Queue via array: maintain `front`, `rear`, `size` (circular array to reuse space).
- Cross-implementation: LIFO from FIFO, or FIFO from LIFO, always costs you something — either push is O(n) or pop is O(n). Pick the cheaper direction based on which op is called more often (usually make push expensive, pop cheap, via "shift everything to one helper structure on pop").
- Min Stack: store pairs (value, current_min_so_far) OR use an auxiliary stack tracking running minimum.

### 4. Representative Problems

#### PROBLEM A: Implement Queue using Stacks (#3)

**Recognition:** "build FIFO using only LIFO primitives (push/pop/top)".

**Approach (plain English):**
Use two stacks, `in` and `out`.
- enqueue(x): push x onto `in`.
- dequeue(): if `out` is empty, pour everything from `in` into `out` (this reverses the order, turning LIFO into FIFO), then pop from `out`.

This way each element is reversed exactly once across its lifetime -> amortized O(1).

**Dry run:** enqueue 1,2,3 then dequeue twice, then enqueue 4, then dequeue.

```
enqueue 1: in=[1]        out=[]
enqueue 2: in=[1,2]      out=[]
enqueue 3: in=[1,2,3]    out=[]

dequeue:  out empty -> pour in->out (reverse)
          in=[]         out=[3,2,1]
          pop out -> returns 1
          in=[]         out=[3,2]

dequeue:  out not empty -> pop -> returns 2
          in=[]         out=[3]

enqueue 4: in=[4]        out=[3]

dequeue:  out not empty -> pop -> returns 3
          in=[4]         out=[]
```
Correct FIFO order: 1,2,3 ✓ (4 will come out after, when out empties again)

**Code:**
```cpp
class MyQueue {
    stack<int> in, out;
public:
    void push(int x) { in.push(x); }

    int pop() {
        move();
        int val = out.top();
        out.pop();
        return val;
    }

    int peek() {
        move();
        return out.top();
    }

    bool empty() { return in.empty() && out.empty(); }

private:
    void move() {
        if (out.empty())
            while (!in.empty()) {
                out.push(in.top());
                in.pop();
            }
    }
};
```

**Time:** push O(1). pop/peek O(1) amortized (each element moved at most once between the two stacks).
**Space:** O(n) for the two stacks.

---

#### PROBLEM B: Implement Min Stack (#8)

**Recognition:** "stack but also support getMin() in O(1)".

**Approach (plain English):**
Naive idea: track a separate `minSoFar` variable — but on pop, if the popped element WAS the min, you don't know the previous min anymore. Fix: instead of storing the raw value, store an *encoded* value when pushing something smaller than current min, so popping can reconstruct the previous min. (Simplest production version: just push pairs (val, minSoFar) onto the same stack — O(1) but 2x space. The encoding trick below is the classic O(1) extra-space variant.)

Encoding trick: maintain `minVal`. When pushing x:
- if stack empty, minVal = x, push x.
- if x >= minVal, push x as-is.
- if x < minVal, push `2*x - minVal` (a value guaranteed < x, signals "this was a new minimum"), then set minVal = x.

On pop, if popped value < minVal -> it was an encoded new-min marker -> recover previous min as `2*minVal - poppedValue`, restore minVal to that.

**Dry run:** push 5, push 3, push 7, getMin, pop, getMin

```
push 5: stack empty -> minVal=5, push 5        stack=[5]            minVal=5
push 3: 3 < 5 -> push (2*3-5)=1, minVal=3      stack=[5,1]          minVal=3
push 7: 7 >= 3 -> push 7                       stack=[5,1,7]        minVal=3
getMin -> minVal = 3

pop: top=7, 7 >= minVal(3) -> normal pop       stack=[5,1]          minVal=3
getMin -> 3 (unchanged, correct: stack values are 5,3)

pop: top=1, 1 < minVal(3) -> was a marker
     recover prevMin = 2*3 - 1 = 5
     minVal = 5                                stack=[5]            minVal=5
getMin -> 5  (correct: only 5 left)
```

**Code:**
```cpp
class MinStack {
    stack<long long> st;
    long long minVal;
public:
    void push(int val) {
        if (st.empty()) { minVal = val; st.push(val); }
        else if (val >= minVal) st.push(val);
        else { st.push(2LL*val - minVal); minVal = val; }
    }
    void pop() {
        long long top = st.top(); st.pop();
        if (top < minVal) minVal = 2*minVal - top;
    }
    int top() {
        long long top = st.top();
        return (top < minVal) ? (int)minVal : (int)top;
    }
    int getMin() { return (int)minVal; }
};
```

**Time:** O(1) for all operations.
**Space:** O(n) for the stack, O(1) extra (the trick avoids a second stack).

### 5. Common Mistakes
- In queue-via-stacks: pouring `in` into `out` on *every* dequeue (not just when `out` is empty) — kills amortized O(1), becomes O(n) every time.
- In Min Stack: forgetting to use `long long`/wider type — `2*val - minVal` can overflow for values near INT_MIN/MAX.
- Confusing which structure should be expensive: for "Stack using Queue" the trick is reversed — you rotate the queue (n-1) times after each push so the newest element ends up at the front, making push O(n) but pop O(1).

### 6. Connects To
This is the "mechanics" layer underneath everything else in this topic. Pattern E (LRU/LFU, Celebrity) reuses the exact same "augment basic structure with extra tracking variable" idea (here: minVal; there: hashmap of pointers). Pattern C's monotonic stack is literally a stack used as in #1/#5, just with a smarter push rule.

---

---

## PATTERN 2 — EXPRESSION CONVERSION (Pre-In-Post-fix)

### 1. Core Intuition
Every arithmetic expression is secretly a binary tree: operators are internal nodes, operands are leaves. Infix, prefix, postfix are just three different ways of writing down that same tree by visiting it in different orders — prefix = (root, left, right), postfix = (left, right, root), infix = (left, root, right). Converting between notations = converting between tree traversal orders, without ever building the tree explicitly.

### 2. Why This Pattern Exists
Computers evaluate postfix/prefix trivially (no precedence rules needed), but humans write infix. Compilers must convert infix -> postfix (or build the AST). This pattern tests whether you can simulate "precedence + associativity + parenthesis nesting" using a single stack, and whether you understand operand-pair combination for the reverse direction.

### 3. Core Template / Technique
Two distinct techniques cover all 6 problems:

**TECHNIQUE 1 — Infix -> Postfix/Prefix (the only "hard" direction)**
Use one stack of operators.
- operand -> output directly.
- `(` -> push.
- `)` -> pop and output until `(` is popped (discard the `(`).
- operator op: while stack top has higher precedence (or equal precedence and left-associative) than op, pop it to output. Then push op.
- at end, pop everything remaining.

For prefix: reverse the input string (swap `(`/`)`), run infix->postfix logic with adjusted associativity (right-to-left for ties), then reverse the result.

**TECHNIQUE 2 — Pre/Post-fix <-> Infix/Pre/Post (the "easy" direction)**
Scan in the *correct order* (postfix: left-to-right, prefix: right-to-left).
- operand -> push onto a stack (as a string).
- operator -> pop two operands (op1, op2 — order matters!), combine into a string per target notation, push result back.
At the end, stack has one element = answer.

### 4. Representative Problems

#### PROBLEM A: Infix to Postfix (#9)

**Recognition:** input has `(`, `)`, operators with mixed precedence, output should have no parentheses, operators come after their operands.

**Approach:** Technique 1. Precedence: `^` > `*`,`/` > `+`,`-`. `^` is right-associative (pop on strictly-greater precedence only); others are left-associative (pop on greater-or-equal).

**Dry run:** infix = `a+b*c-d`

```
char | action                                    | stack   | output
-----|--------------------------------------------|---------|--------
a    | operand -> output                          | []      | a
+    | stack empty -> push                         | [+]     | a
b    | operand -> output                          | [+]     | ab
*    | prec(*) > prec(+) -> push                   | [+,*]   | ab
c    | operand -> output                          | [+,*]   | abc
-    | prec(-) <= prec(*) -> pop * ; prec(-)<=prec(+) -> pop + ; push -
                                                    | [-]     | abc*+
d    | operand -> output                          | [-]     | abc*+d
end  | pop remaining                              | []      | abc*+d-
```
Result: `abc*+d-`  (matches expected: a+(b*c) computed first, then minus d)

**Code:**
```cpp
int prec(char c) {
    if (c=='^') return 3;
    if (c=='*'||c=='/') return 2;
    if (c=='+'||c=='-') return 1;
    return -1;
}

string infixToPostfix(string s) {
    stack<char> st;
    string res;
    for (char c : s) {
        if (isalnum(c)) res += c;
        else if (c=='(') st.push(c);
        else if (c==')') {
            while (st.top()!='(') { res += st.top(); st.pop(); }
            st.pop();
        } else {
            while (!st.empty() && st.top()!='(' &&
                   (prec(st.top()) > prec(c) ||
                    (prec(st.top())==prec(c) && c!='^'))) {
                res += st.top(); st.pop();
            }
            st.push(c);
        }
    }
    while (!st.empty()) { res += st.top(); st.pop(); }
    return res;
}
```

**Time:** O(n) — each char pushed/popped at most once.
**Space:** O(n) for stack + output.

---

#### PROBLEM B: Postfix to Infix (#13)

**Recognition:** input has operators after operands, no parentheses; output needs full parenthesization showing grouping.

**Approach:** Technique 2, left-to-right scan. On operator, pop two strings (op2 first since it's on top, then op1), wrap as `(op1 OPERATOR op2)`, push back.

**Dry run:** postfix = `ab+c*` (means (a+b)*c)

```
char | action                                      | stack
-----|-----------------------------------------------|------------------
a    | operand -> push "a"                           | ["a"]
b    | operand -> push "b"                           | ["a","b"]
+    | pop "b"(op2), pop "a"(op1) -> "(a+b)" -> push  | ["(a+b)"]
c    | operand -> push "c"                           | ["(a+b)","c"]
*    | pop "c"(op2), pop "(a+b)"(op1) -> "((a+b)*c)" -> push | ["((a+b)*c)"]
```
Result: `((a+b)*c)`

**Code:**
```cpp
string postfixToInfix(string s) {
    stack<string> st;
    for (char c : s) {
        if (isalnum(c)) st.push(string(1,c));
        else {
            string op2 = st.top(); st.pop();
            string op1 = st.top(); st.pop();
            st.push("(" + op1 + c + op2 + ")");
        }
    }
    return st.top();
}
```

**Time:** O(n * L) where L is growing string length due to concatenation (effectively O(n^2) worst case for string building, O(n) conceptually for stack ops).
**Space:** O(n) for stack of strings.

### 5. Common Mistakes
- Swapping op1/op2 order — postfix `ab-` means `a-b`, but you pop b first then a. Pushing `op2 OPERATOR op1` instead of `op1 OPERATOR op2` silently breaks non-commutative operators (`-`, `/`).
- For prefix conversions: forgetting to reverse the string AND swap `(`<->`)` before applying infix logic, or forgetting to reverse the *result* afterward.
- `^` (power) associativity: treating it as left-associative breaks `a^b^c` (should be `a^(b^c)`).
- Treating multi-digit numbers / multi-char identifiers as single characters — real inputs may need tokenization, though sheet problems usually assume single-char operands.

### 6. Connects To
The "pop two, combine, push back" mechanic in Technique 2 reappears conceptually in Pattern 5 (combining state). The single-operator-stack-with-precedence idea (Technique 1) is the direct ancestor of evaluating expressions — and the general "maintain a stack while scanning left-to-right, popping when a condition is violated" is exactly the skeleton of Pattern 3's monotonic stack, just with a different popping condition (precedence vs. value comparison).

---

---

## PATTERN 3 — MONOTONIC STACK (core)

### 1. Core Intuition
Imagine people standing in a line, each with a height, all facing right. You walk along the line holding a stack of people who are "still waiting to find someone taller than them." When you meet someone taller than the person on top of your stack, that topmost person finally found their answer — pop them, record the answer, and check the new top against the same new person. Anyone never popped just never finds an answer.

### 2. Why This Pattern Exists
Brute force for "nearest greater/smaller element" is O(n^2) — for each element, scan all others. The monotonic stack exploits a key fact: if A is shorter than B and B comes before some taller C, A's answer is also satisfied at C or earlier — so once we know B's relationship to C, A doesn't need to be rechecked independently. This collapses redundant comparisons, giving O(n) total because each element is pushed once and popped at most once.

### 3. Core Template / Technique
```
stack = []   (stores indices)
for i in range(n):
    while stack not empty and condition(arr[stack.top()], arr[i]) is violated:
        idx = stack.pop()
        answer[idx] = i   (or arr[i], or i - stack.top() - 1, etc.)
    stack.push(i)
# anything left in stack at the end has "no answer" (e.g. -1 or n)
```
The "condition" and what you compute on pop are the only things that change across all 12 problems:
- Next Greater Element: pop while `arr[top] < arr[i]` -> answer[popped] = arr[i]
- Next Smaller Element: pop while `arr[top] > arr[i]` -> answer[popped] = arr[i]
- Histogram area: pop while `height[top] >= height[i]` -> width = i - stack.top() - 1 (or i if empty)
- For "circular" versions (NGE-II): run the array twice (i from 0 to 2n-1, use `i % n`).
- For "contribution" problems (subarray min/range sum): on pop, the popped element's "span of influence" is `(i - prevStackTop - 1) * (popIdx - prevStackTop - 1)` style range counting — same skeleton, different arithmetic on pop.

### 4. Representative Problems

#### PROBLEM A: Next Greater Element (#15)

**Recognition:** "for each element, find the first element to its right that is strictly greater; if none, -1".

**Approach (plain English):**
Traverse right-to-left. Maintain a stack of "candidates that could be the next greater element for something to their left," kept in decreasing order from bottom to top. For each element, pop everything smaller-or-equal (current is closer and bigger, so it dominates). Whatever remains on top after popping is the NGE for current. Then push current.

**Dry run:** arr = [2, 1, 2, 4, 3], find NGE for each (right-to-left)

```
i=4, arr=3: stack=[]               -> NGE[4]=-1   push 3 -> stack=[3]
i=3, arr=4: pop 3 (3<=4)           -> stack=[]
            stack empty            -> NGE[3]=-1   push 4 -> stack=[4]
i=2, arr=2: top=4, 4>2, no pop     -> NGE[2]=4     push 2 -> stack=[4,2]
i=1, arr=1: top=2, 2>1, no pop     -> NGE[1]=2     push 1 -> stack=[4,2,1]
i=0, arr=2: pop 1 (1<=2)           -> stack=[4,2]
            top=2, 2<=2, pop 2     -> stack=[4]
            top=4, 4>2, no pop     -> NGE[0]=4     push 2 -> stack=[4,2]
```
Result: NGE = [4, 2, 4, -1, -1]

**Code:**
```cpp
vector<int> nextGreaterElement(vector<int>& arr) {
    int n = arr.size();
    vector<int> nge(n, -1);
    stack<int> st;  // stores values
    for (int i = n-1; i >= 0; i--) {
        while (!st.empty() && st.top() <= arr[i]) st.pop();
        if (!st.empty()) nge[i] = st.top();
        st.push(arr[i]);
    }
    return nge;
}
```

**Time:** O(n) — each element pushed once, popped at most once.
**Space:** O(n) for stack + answer array.

---

#### PROBLEM B: Largest Rectangle in Histogram (#25)

**Recognition:** array of bar heights, find max-area rectangle that fits entirely within consecutive bars — classic "for each bar, find how far left and right it can extend at its own height" -> nearest smaller element on both sides.

**Approach (plain English):**
For each bar i, the maximal rectangle using height[i] extends from the nearest bar on the left that is shorter than height[i], to the nearest bar on the right that is shorter. Width = (right_boundary - left_boundary - 1). Compute area = height[i] * width, take max over all i. Both boundaries come from "nearest smaller element" — solvable with one pass using a monotonic increasing stack, computing left and right boundary together.

Single-pass trick: traverse left-to-right with a stack kept increasing. When height[i] < height[stack.top()], pop and finalize that popped bar's rectangle immediately — left boundary = new stack top (or -1), right boundary = i.

**Dry run:** heights = [2, 1, 5, 6, 2, 3]

```
i=0,h=2: stack empty -> push 0          stack=[0]
i=1,h=1: h[0]=2 > 1 -> pop 0
         left = -1 (stack empty), right = 1
         width = 1-(-1)-1 = 1, area = 2*1 = 2   -> maxArea=2
         push 1                                  stack=[1]
i=2,h=5: h[1]=1 <=5 -> push 2            stack=[1,2]
i=3,h=6: h[2]=5 <=6 -> push 3            stack=[1,2,3]
i=4,h=2: h[3]=6 > 2 -> pop 3
         left=2, right=4, width=4-2-1=1, area=6*1=6 -> maxArea=6
         h[2]=5 > 2 -> pop 2
         left=1, right=4, width=4-1-1=2, area=5*2=10 -> maxArea=10
         h[1]=1 <=2 -> push 4                    stack=[1,4]
i=5,h=3: h[4]=2 <=3 -> push 5            stack=[1,4,5]
end: pop remaining, right=n=6
  pop 5: left=4, width=6-4-1=1, area=3*1=3
  pop 4: left=1, width=6-1-1=4, area=2*4=8
  pop 1: left=-1, width=6-(-1)-1=6, area=1*6=6
maxArea remains 10
```
Result: 10 (the 5,6 bars forming a 2-wide x 5-tall rectangle)

**Code:**
```cpp
int largestRectangleArea(vector<int>& h) {
    int n = h.size(), maxArea = 0;
    stack<int> st;  // indices, increasing heights
    for (int i = 0; i <= n; i++) {
        int curH = (i == n) ? 0 : h[i];
        while (!st.empty() && h[st.top()] >= curH) {
            int height = h[st.top()]; st.pop();
            int left = st.empty() ? -1 : st.top();
            int width = i - left - 1;
            maxArea = max(maxArea, height * width);
        }
        st.push(i);
    }
    return maxArea;
}
```

**Time:** O(n) — same amortized argument, each index pushed/popped once.
**Space:** O(n) for stack.

### 5. Common Mistakes
- Strict vs non-strict comparison (`<` vs `<=`): for NGE you want strictly greater, so pop condition must be `<=` (pop equal elements too, since current is closer). Getting this backwards gives wrong answers on duplicate values.
- In histogram, forgetting the sentinel `i == n` with `curH = 0` — without it, bars still in the stack at the end never get their area computed.
- Confusing "what you push" — push indices when you need positions (for width/distance calculations), push values when you only need the value itself (plain NGE).
- For circular array problems (NGE-II), iterating `2n` times but forgetting to only *push* during the first pass (or guard against double-pushing), which corrupts results.
- Maximal Rectangle (matrix version) — forgetting it's just "Largest Rectangle in Histogram" applied row-by-row, where each row's histogram is built by accumulating consecutive 1s vertically.

### 6. Connects To
The popping condition here is a direct generalization of the precedence-popping in Pattern 2 — both are "pop while a comparison fails, because the new element invalidates old stack entries." Pattern 4 (monotonic deque) is this exact same idea but with eviction from both ends due to a sliding window constraint. Many "contribution" problems (subarray min/range sums) literally reuse the histogram boundary-finding code, just multiplying by different quantities on pop.

---

---

## PATTERN 4 — MONOTONIC DEQUE (Sliding Window Maximum)

### 1. Core Intuition
Picture a line of people again, but now you're only allowed to look at a "window" of k consecutive people at a time, and the window slides forward one person at a time. You want the tallest person currently visible, at every position of the window. Keep a guest-list (deque) of candidates for "tallest in current or future window," ordered from tallest (front) to shortest (back) — anyone shorter than a newer arrival is permanently useless and gets kicked out from the back.

### 2. Why This Pattern Exists
This is Pattern 3's monotonic stack with one extra constraint: elements also expire when they fall outside the window's left edge. A single stack only handles eviction from one end (the top, due to value comparison); here you need eviction from the back due to value comparison AND eviction from the front due to position/age. That's exactly what a deque (double-ended queue) gives you.

### 3. Core Template / Technique
```
deque = []   (stores indices, values decreasing from front to back)
for i in range(n):
    # remove indices out of window from the front
    while deque not empty and deque.front() <= i - k:
        deque.popFront()
    # remove smaller elements from the back (they're useless now)
    while deque not empty and arr[deque.back()] <= arr[i]:
        deque.popBack()
    deque.pushBack(i)
    if i >= k-1:
        answer.append(arr[deque.front()])
```

### 4. Representative Problem (only 1 problem in the sheet)

#### PROBLEM: Sliding Window Maximum (#27)

**Recognition:** "return the maximum of every contiguous subarray of size k" — fixed window size, need running max (or min).

**Approach (plain English):**
Maintain a deque of indices such that `arr[deque]` is strictly decreasing front-to-back. The front is always the index of the current window's maximum. Before processing index i: drop the front if it has slid out of the window (`front <= i-k`). Then drop from the back any index whose value is <= arr[i] (they can never be the max again once arr[i] is in the window, since arr[i] is both bigger and more recent). Push i to the back. Once i >= k-1, the front holds the answer for this window.

**Dry run:** arr = [1,3,-1,-3,5,3,6,7], k = 3

```
i=0,val=1: deque=[]                          -> push 0          deque=[0]
i=1,val=3: back arr[0]=1<=3 -> pop 0          deque=[]
           push 1                            deque=[1]
i=2,val=-1: back arr[1]=3>-1, no pop
            push 2                           deque=[1,2]
            i>=k-1(2>=2) -> ans: arr[front=1]=3   ans=[3]
i=3,val=-3: front=1, 1<=3-3=0? no
            back arr[2]=-1>-3, no pop
            push 3                           deque=[1,2,3]
            ans: arr[front=1]=3                  ans=[3,3]
i=4,val=5:  front=1, 1<=4-3=1? yes -> pop front 1   deque=[2,3]
            back arr[3]=-3<=5 -> pop 3        deque=[2]
            back arr[2]=-1<=5 -> pop 2        deque=[]
            push 4                           deque=[4]
            ans: arr[front=4]=5                  ans=[3,3,5]
i=5,val=3:  front=4, 4<=5-3=2? no
            back arr[4]=5>3, no pop
            push 5                           deque=[4,5]
            ans: arr[front=4]=5                  ans=[3,3,5,5]
i=6,val=6:  front=4, 4<=6-3=3? no
            back arr[5]=3<=6 -> pop 5         deque=[4]
            back arr[4]=5<=6 -> pop 4         deque=[]
            push 6                           deque=[6]
            ans: arr[front=6]=6                  ans=[3,3,5,5,6]
i=7,val=7:  front=6, 6<=7-3=4? no
            back arr[6]=6<=7 -> pop 6         deque=[]
            push 7                           deque=[7]
            ans: arr[front=7]=7                  ans=[3,3,5,5,6,7]
```
Result: [3,3,5,5,6,7] ✓

**Code:**
```cpp
vector<int> maxSlidingWindow(vector<int>& arr, int k) {
    deque<int> dq;  // stores indices, decreasing values front-to-back
    vector<int> ans;
    int n = arr.size();
    for (int i = 0; i < n; i++) {
        if (!dq.empty() && dq.front() <= i - k) dq.pop_front();
        while (!dq.empty() && arr[dq.back()] <= arr[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1) ans.push_back(arr[dq.front()]);
    }
    return ans;
}
```

**Time:** O(n) — each index pushed once, popped at most once (front or back).
**Space:** O(k) for the deque.

### 5. Common Mistakes
- Using `<` instead of `<=` when popping from the back on ties — with `<`, duplicate max values can cause stale (already-expired) indices to linger at the front, giving wrong answers.
- Checking the front-expiry condition AFTER the back-popping loop instead of before — order matters when k=1 or when the front element is also the one that should be popped from the back.
- Forgetting that the deque stores indices, not values — you need indices to detect window expiry.
- Off-by-one on `i - k`: the correct condition is `front_index <= i - k` (strictly less than the window's left edge `i-k+1`).

### 6. Connects To
This is Pattern 3 with an added "age limit" — the back-eviction logic (`arr[back] <= arr[i]` -> pop) is character-for-character identical to the monotonic stack's popping condition. If you ever need "sliding window minimum," just flip the comparison. The general lesson: whenever a monotonic structure needs to also respect a positional/time window, reach for a deque instead of a stack.

---

---

## PATTERN 5 — STACK/QUEUE-POWERED DESIGN

### 1. Core Intuition
These are "build a small machine" problems. The Celebrity Problem is like a knockout tournament: pit two candidates against each other, the loser is eliminated, the survivor faces the next challenger — one pass, one survivor remains, then verify. LRU/LFU Cache are like a library shelf: most-recently-borrowed books sit at the front (or get a higher "recency stamp"); when the shelf is full, you remove from the back (or lowest-frequency end).

### 2. Why This Pattern Exists
Real systems need O(1) lookups (hashmap) but also need ordering information (recency, frequency) that a plain hashmap can't give you. The pattern tests whether you can combine a hashmap (for O(1) access) with a structure that maintains order (doubly linked list for recency, or elimination logic for celebrity) — a recurring "two data structures working together" theme.

### 3. Core Template / Technique
- Celebrity: two-pointer elimination using the knows(a,b) relation — O(n) instead of O(n^2) checking all pairs.
- LRU Cache: hashmap<key, Node*> + doubly linked list ordered by recency (head = most recent, tail = least recent). get/put both move the node to the head; eviction removes the tail.
- LFU Cache: hashmap<key, Node> + hashmap<freq, doubly linked list of nodes at that frequency> + track minFreq. On access, move node from freq f's list to freq f+1's list; eviction removes from minFreq's list tail.

### 4. Representative Problems

#### PROBLEM A: The Celebrity Problem (#28)

**Recognition:** n x n matrix M where M[i][j]=1 means "i knows j". Celebrity = someone everyone else knows, but who knows nobody. Find them in O(n), not O(n^2).

**Approach (plain English):**
Use two pointers, top=0, bottom=n-1. Repeatedly compare: does `top` know `bottom`?
- If yes -> `top` cannot be the celebrity (celebrity knows nobody) -> advance top.
- If no -> `bottom` cannot be the celebrity (everyone must know the celebrity) -> decrement bottom.

This eliminates one candidate per comparison; after the loop, one candidate remains. Verify it: check its entire row is all 0 (knows nobody) and its entire column is all 1 except itself (everyone knows them).

**Dry run:** M (1=knows) =
```
      0  1  2  3
   0 [0, 0, 1, 0]
   1 [0, 0, 1, 0]
   2 [0, 0, 0, 0]
   3 [0, 0, 1, 0]
```
(Person 2 is the celebrity: row 2 is all 0, column 2 is all 1 except M[2][2])

```
top=0, bottom=3
M[0][3]=0 -> 0 doesn't know 3 -> 3 can't be celebrity -> bottom=2
top=0, bottom=2
M[0][2]=1 -> 0 knows 2 -> 0 can't be celebrity -> top=1
top=1, bottom=2
M[1][2]=1 -> 1 knows 2 -> 1 can't be celebrity -> top=2
top=2, bottom=2  -> top==bottom, stop. candidate=2

Verify candidate=2:
  row 2 = [0,0,0,0] -> all 0 ✓ (knows nobody)
  column 2 = [1,1,0,1] -> all 1 except M[2][2] ✓ (everyone knows them)
candidate 2 is the celebrity.
```

**Code:**
```cpp
int celebrity(vector<vector<int>>& M, int n) {
    int top = 0, bottom = n - 1;
    while (top < bottom) {
        if (M[top][bottom] == 1) top++;   // top knows bottom -> top eliminated
        else bottom--;                     // top doesn't know bottom -> bottom eliminated
    }
    int candidate = top;
    for (int i = 0; i < n; i++) {
        if (i == candidate) continue;
        if (M[candidate][i] != 0 || M[i][candidate] != 1)
            return -1;  // no celebrity
    }
    return candidate;
}
```

**Time:** O(n) for elimination + O(n) for verification = O(n).
**Space:** O(1) extra.

---

#### PROBLEM B: LRU Cache (#29)

**Recognition:** design get(key)/put(key,val) both in O(1), with a fixed capacity; when full, evict the least-recently-used item.

**Approach (plain English):**
Doubly linked list ordered by recency: head-side = most recently used, tail-side = least recently used. Hashmap maps key -> node pointer for O(1) lookup. On get(key): if present, unlink the node and re-insert it right after head (mark as most recent), return its value. On put(key,val): if key exists, update value and move to front. If new and at capacity, remove the node just before tail (the LRU victim) from both list and hashmap. Insert new node right after head.

**Dry run:** capacity=2
```
put(1,1): list: head <-> [1:1] <-> tail        map={1:node1}
put(2,2): list: head <-> [2:2] <-> [1:1] <-> tail   map={1:n1,2:n2}
get(1)  : found -> move [1:1] to front
          list: head <-> [1:1] <-> [2:2] <-> tail   return 1
put(3,3): capacity full -> evict node before tail = [2:2]
          list: head <-> [3:3] <-> [1:1] <-> tail   map={1:n1,3:n3}
get(2)  : not in map -> return -1
```

**Code:**
```cpp
class LRUCache {
    struct Node {
        int key, val;
        Node* prev; Node* next;
        Node(int k, int v): key(k), val(v), prev(nullptr), next(nullptr) {}
    };
    int cap;
    unordered_map<int, Node*> mp;
    Node *head, *tail;  // dummy sentinels

    void remove(Node* n) {
        n->prev->next = n->next;
        n->next->prev = n->prev;
    }
    void insertFront(Node* n) {
        n->next = head->next;
        n->prev = head;
        head->next->prev = n;
        head->next = n;
    }

public:
    LRUCache(int capacity) : cap(capacity) {
        head = new Node(-1,-1);
        tail = new Node(-1,-1);
        head->next = tail;
        tail->prev = head;
    }

    int get(int key) {
        if (mp.find(key) == mp.end()) return -1;
        Node* n = mp[key];
        int val = n->val;
        remove(n);
        insertFront(n);
        return val;
    }

    void put(int key, int value) {
        if (mp.find(key) != mp.end()) {
            Node* n = mp[key];
            n->val = value;
            remove(n);
            insertFront(n);
            return;
        }
        if ((int)mp.size() == cap) {
            Node* lru = tail->prev;
            remove(lru);
            mp.erase(lru->key);
            delete lru;
        }
        Node* n = new Node(key, value);
        mp[key] = n;
        insertFront(n);
    }
};
```

**Time:** O(1) for get and put.
**Space:** O(capacity) for the list + hashmap.

### 5. Common Mistakes
- Celebrity: comparing `M[top][bottom]` direction incorrectly — if top knows bottom, it's `top` that's eliminated (celebrity can't know anyone), not bottom. Mixing this up inverts the whole logic.
- Celebrity: skipping the verification step — the two-pointer pass only finds a *candidate*; if no celebrity exists, you must still return -1.
- LRU: forgetting to update the hashmap when evicting (memory leak / stale pointer) or forgetting `delete` causing leaks in C++.
- LRU: updating value on an existing key but forgetting to also move it to front — breaks recency ordering.
- LFU (same family): forgetting to update `minFreq` when the only node at the current minFreq moves to a higher frequency bucket — eviction will then target the wrong bucket.

### 6. Connects To
This pattern is the "capstone" — it combines Pattern 1's "augment a basic structure with an auxiliary tracker" (minVal in Min Stack -> hashmap+pointers here) with general two-pointer elimination ideas you'll see again in arrays/strings. LRU's doubly-linked-list-as-queue-with-random-access is itself a hybrid: FIFO eviction order (queue-like) but O(1) arbitrary removal (linked-list power that a plain array/queue lacks).
