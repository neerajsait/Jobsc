# Job Shop Scheduler: A Beginner's Guide

This guide is written so a beginner can follow it. Work with your AI agent one step at a time, and run the code after each step.

## What we are building

A Python program that:
1. Reads a list of jobs (each job has steps called operations).
2. Each operation needs one machine for some time.
3. Finds the order that finishes everything as early as possible. That finish time is called the makespan.

**Two rules:**
- The operations of one job must run in order.
- A machine can do only one operation at a time.

## Words to know

| Word | Simple meaning |
|---|---|
| **Operation** | One step of a job, e.g. "use Machine 1 for 3 hours" |
| **Job** | A list of operations in order |
| **Makespan** | Time when the last job finishes |
| **Node** | One half-finished plan (some operations placed, some not) |
| **Branch** | Make new nodes by placing one more operation |
| **Bound** | A guess of the best finish time a node could still reach |
| **Prune** | Throw a node away because its bound is not better than the best plan we already have |

## Project structure

```text
job_shop_scheduler/
├── main.py                  # run the program from here
├── data/
│   └── sample_jobs.json     # example jobs
└── scheduler/
    ├── __init__.py
    ├── models.py            # Operation, Job, Machine, Schedule classes
    ├── loader.py            # reads the JSON file
    ├── base.py              # Scheduler (abstract parent class)
    ├── greedy.py            # quick, simple solver
    ├── bounds.py            # lower_bound function
    ├── branch_and_bound.py  # the main solver
    ├── parallel.py          # runs the search on many CPU cores
    ├── utils.py             # timer decorator, worker pool helper
    └── display.py           # prints the schedule
```

## Steps

**Give the agent one step at a time. Never ask for the whole project at once.**

### Step 0: Setup
- Install Python 3.12 or newer.
- Create the folders and empty files shown above.
- Make a virtual environment (ask the agent: *"How do I create and use a Python virtual environment?"*).

**Ask the agent:** *"I am a beginner. Help me create this folder structure for a Python project and explain what each file is for."*

### Step 1: The data classes (`models.py`)
Make four small classes with type hints:
- `Operation`: job_id, machine_id, duration (use a dataclass, make it frozen).
- `Job`: has a list of Operations (a Job has Operations).
- `Machine`: remembers when it is free.
- `Schedule`: remembers the start time of every operation and can give the makespan.

**Ask the agent:** *"Write Operation, Job, Machine and Schedule classes in models.py using dataclasses and type hints. Keep them simple and add comments in easy English."*

### Step 2: Load the data (`loader.py` and `sample_jobs.json`)
Use this small example. Each pair is (machine, time).
- Job 1: (M1,3) (M2,2) (M3,2)
- Job 2: (M2,2) (M3,1) (M1,4)
- Job 3: (M3,4) (M1,3)

**Ask the agent:** *"Put these 3 jobs in a JSON file and write a function that reads it and returns a list of Job objects."*

### Step 3: Makespan and rule check (`base.py`, `models.py`)
- Write a pure function `makespan(schedule)`. A pure function only uses its inputs and changes nothing outside.
- Make an abstract class `Scheduler` with one method: `solve(jobs) -> Schedule`. Every solver will inherit from it.

**Ask the agent:** *"Write an abstract base class Scheduler with a solve method. Also write a pure makespan function. Explain what 'abstract' means."*

### Step 4: Greedy solver (`greedy.py`)
A greedy solver always takes the best-looking choice right now, for example "do the shortest ready operation first". It is fast but not always the best. We use it for one reason: it gives us a first answer to beat.

**Ask the agent:** *"Write GreedyScheduler that inherits Scheduler. Always place the ready operation with the shortest duration. Return a Schedule."*

*Check your work: run it on the sample data and print the makespan. Write it down.*

### Step 5: The lower bound (`bounds.py`)
A lower bound is a number the final makespan can never beat. Use the bigger of these two:
- For each job: time already used plus time of its remaining operations.
- For each machine: its free time plus the time of all operations still waiting for it.

Make it a pure function. It must never guess too high, or we could throw away the best answer.

**Ask the agent:** *"Write lower_bound(state) as a pure function. Explain in simple words why it never gives a number that is too big."*

### Step 6: Branch-and-bound solver (`branch_and_bound.py`)
This is the main part. The idea in plain steps:
1. Start with the greedy makespan as the best so far.
2. Put the empty plan in a heap (Python's `heapq`). The heap always gives you the node with the smallest bound first.
3. Take a node out. If its bound is not better than the best so far, throw it away (prune).
4. If it is complete, it is the new best. Save it.
5. Otherwise make child nodes by placing the next operation of each job. Use a generator (`yield`) to create the children one by one.
6. Repeat until the heap is empty. The best plan is the answer.

**Ask the agent:** *"Write BranchAndBound that inherits Scheduler. Use heapq, the greedy answer as the first best value, lower_bound for pruning, and a generator that yields child nodes. Add comments for each step."*

*Check your work: the result must not be worse than greedy. For the sample jobs, the best makespan should be 10.*

### Step 7: Helper tools (`utils.py`)
- A decorator `@timed` that prints how long a function took.
- A context manager (`with` block) that opens a worker pool and always closes it.

**Ask the agent:** *"Write a timed decorator and a context manager for a multiprocessing pool. Explain how each one works in simple words."*

### Step 8: Run in parallel (`parallel.py`)
The search uses the CPU a lot (it is CPU-bound). Python threads cannot run CPU work at the same time because of the GIL, so we use multiprocessing, which runs separate processes.
- Split the first choices (top-level branches) between the worker processes.
- Share the best makespan found so far, so every worker can prune with it.

**Ask the agent:** *"Add a parallel version using multiprocessing.Pool. Split the top-level branches between workers and share the best value. Explain why we use multiprocessing and not threading."*

### Step 9: Show the result and run it (`display.py`, `main.py`)
- Print a simple text chart: each machine, then the operations with start and end times.
- `main.py`: load jobs, run greedy, run branch-and-bound, run parallel, print the makespan and time for each.

**Ask the agent:** *"Write display.py to print each machine's schedule as text. Then write main.py that runs all three solvers and prints the makespan and time of each."*

### Step 10: Use functional style
Go back through the code and use:
- `map`, `filter`, `max`, `functools.reduce` where they make code shorter.
- List comprehensions instead of small `for` loops.
- Immutable states (`tuple`, frozen `dataclass`) so they are safe to pass between processes.

**Ask the agent:** *"Review my code and show where I can use comprehensions, map, filter and reduce. Explain each change."*

## Rules for working with the AI agent
1. **Write the plan first.** Tell the agent the rules and what you want before asking for code.
2. **One step at a time.** Paste the code from earlier steps so the agent knows the class names.
3. **Read every line.** Ask "explain this line in simple words" until you understand it.
4. **Run the code after each step.** If it fails, paste the full error message to the agent.
5. **Ask for small functions.** Short code is easier to read and fix.
