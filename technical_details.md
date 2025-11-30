# TECHNICAL DOCUMENTATION – SCHEDULING APP  

Lean but complete; includes all technical details discussed.

---

# 1. PROBLEM DEFINITION & SCOPE

The goal is to build a **resource allocation + scheduling engine** that solves problems like:

- School timetables (teachers, subjects, rooms, periods)
- Avoiding teacher overload (no 4 consecutive classes)
- Avoiding subject clustering (no 3 geography classes in a row)
- Ensuring feasible subject frequency (e.g., 3 science classes/week)
- Room/teacher availability constraints

Later expansion: general scheduling across ANY domain (e.g., developer-task assignment, delivery routing, workforce scheduling).

Important: constraints differ wildly across domains, so the app must eventually support user-defined entities + constraints.

---

# 2. NON‑ML SCHEDULING METHODS (AND WHY THEY ARE NOT ML)

Scheduling requires **Operations Research (OR)** methods, not ML.

## 2.1 ILP (Integer Linear Programming)

Variables are integers; constraints are linear. Good for smaller timetable optimizations.

## 2.2 Constraint Programming (CP)

User expresses constraints; solver searches systematically. Very strong for timetables.

## 2.3 Heuristics (GA, Simulated Annealing, Local Search)

Search-based approaches that iteratively improve solutions. Useful for large search spaces.

## 2.4 CP-SAT (OR-Tools’ solver)

Google’s hybrid SAT + CP + MILP engine. State-of-the-art. Solves large and complex constraint problems very efficiently.

## 2.5 Why these are NOT ML

- They **do not learn** from user behavior.
- They simply **solve** a constraint satisfaction problem.
- You explicitly define constraints → solver finds a valid configuration.

ML becomes relevant **after** deployment when user patterns exist.

---

# 3. WHY ML COMES *AFTER* A NON‑ML SOLVER

ML requires:

- Historical timetables
- User overrides (what they changed)
- Teacher preference patterns
- Constraints that frequently break
- How often admins move subjects/slots

ML contributes only after this data exists.

### ML improves:

- Soft constraint weights
- Predicted teacher fatigue
- Predicted “pleasantness” of a timetable
- Suggesting subject distribution
- Detecting patterns (“avoid back-to-back heavy subjects”)

ML does **NOT** replace OR-Tools; it **augments** the objective.

---

# 4. OR-TOOLS – WHAT IT GIVES YOU & WHAT IT REPLACES

Google's OR-Tools is open-source, importable in Python, and provides:

- CP-SAT solver (world-class)
- Linear solvers
- Routing solvers
- Search heuristics
- Automatic backtracking
- Constraint propagation
- Parallel search
- Conflict-driven clause learning

## 4.1 What you would otherwise need to build manually

Without OR-Tools you’d need to build:

- Backtracking search
- Constraint propagation system
- Optimization objective engine
- Heuristic initializers
- Branch-and-bound pruning
- Restart strategies
- Feasibility detection engine
- Multi-threaded search

This is **years** of work and still inferior.

## 4.2 Example: custom solver vs OR-Tools

### Your custom naive solver (bad):

```python
for slot in slots:
    for teacher in teachers:
        if teacher_available(teacher, slot):
            assign(...)
        if violates_constraints():
            backtrack()
```

Slow, exponential, brittle.

### OR-Tools version (good):

```python
from ortools.sat.python import cp_model

model = cp_model.CpModel()

x = model.NewBoolVar("math_period_1")
model.Add(x == 1)

solver = cp_model.CpSolver()
solver.Solve(model)
```

Everything else → OR-Tools handles.

---

# 5. APP ARCHITECTURE

## 5.1 High‑level architecture

```
Frontend UI 
 → Backend API (FastAPI/Node) 
   → Constraint Builder 
     → OR-Tools CP-SAT Solver 
       → JSON Output (Timetable)
```

## 5.2 Data modeling

You must define:

- Entities: teachers, rooms, subjects, students/groups, time slots
- Variables: x[teacher, class, slot] = 0/1
- Hard constraints:
  - Teacher availability
  - No room clashes
  - Subject must appear N times per week
  - Max consecutive periods
- Soft constraints:
  - Avoid clustering
  - Spread subjects across days
  - Teacher preference times
  - Smooth load distribution

## 5.3 Diagnostics engine

Returns WHY a schedule fails:

- Resource shortage
- Constraint contradictions
- Overloaded teacher/room
- Impossible frequency requirements

This is a major differentiator.

## 5.4 Natural‑language constraint input

LLM interprets text:

> “Avoid math after lunch and give Teacher A one free period daily.”

Converts to constraint objects to feed into OR-Tools.

---

# 6. ML LAYER (AFTER V1)

ML becomes useful ONLY after enough user behavior exists.

## 6.1 When ML becomes applicable

After dozens of organizations generate:

- Completed schedules
- Manual overrides
- Teacher complaints/preferences
- Admin modifications
- History of infeasible constraint combinations

## 6.2 What ML can do

- Predict ideal distribution patterns
- Predict fatigue or workload imbalance
- Predict teacher slot preferences
- Score timetable “pleasantness”
- Identify common error patterns
- Suggest constraint relaxations

## 6.3 What ML cannot do

- Replace OR-Tools
- Guarantee feasibility
- Magically solve scheduling problems with no constraints

---

# 7. CROSS‑DOMAIN GENERALIZATION

You asked whether the app can support ANY domain with ANY variables.

## 7.1 Technically possible, but very complex

Different domains have different:

- Time models  
- Resource models  
- Constraint types  
- Optimization goals

## 7.2 You will need:

- A universal entity model:
  - Resource
  - Task
  - Skill/capacity
  - Time unit
- Constraint templates:
  - Max consecutive tasks
  - Must occur N times/week
  - “Exactly one resource per task”
  - “Minimize total cost”
- A mini DSL for constraints
- Natural-language-to-constraint compiler

## 7.3 Realistic roadmap

1. Start with school timetables  
2. Add workforce scheduling  
3. Add developer task assignment  
4. Add modeling templates  
5. Add natural-language constraint builder  

This sequential growth keeps complexity manageable.

---

# END OF TECHNICAL DOCUMENT
