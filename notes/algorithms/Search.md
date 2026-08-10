# Overview

- Search algorithms explore a state space to find a goal state, path, assignment, or high-quality solution

- A search problem contains:

    State: One configuration of the problem

    Initial state: Starting configuration

    Actions/successors: Legal next states

    Goal test: Determines success

    Step cost: Cost of one transition

    Path cost `g(n)`: Total cost from the start to node `n`

- Search appears in pathfinding, games, scheduling, constraint satisfaction, routing, planning, theorem proving, and combinatorial optimization

- Important measurements:

    Completeness: Guaranteed to find a solution when one exists

    Optimality: Guaranteed to return the best solution

    Time complexity: Expanded/generated work

    Space complexity: Memory retained

    Solution quality: Objective value or approximation gap

# Graph vs Tree Search

- Tree search may generate the same state through many paths

- Graph search tracks visited states or best-known costs to avoid useless repetition

- A node usually stores state, parent, action, path cost, depth, and optional heuristic values

- Frontier/open set contains generated nodes waiting to expand

- Explored/closed set contains states already processed under the algorithm's policy

- Duplicate detection must use a stable state identity. Mutable state objects make unsafe hash keys

- Cycle checking only on the current path prevents loops but does not prevent duplicates reached by different branches

# Breadth-First Search

- BFS expands the shallowest nodes first with a FIFO queue

    ```python
    from collections import deque


    def bfs(start, is_goal, neighbors):
        frontier = deque([start])
        parent = {start: None}

        while frontier:
            state = frontier.popleft()

            if is_goal(state):
                return reconstruct(parent, state)

            for neighbor in neighbors(state):
                if neighbor not in parent:
                    parent[neighbor] = state
                    frontier.append(neighbor)

        return None
    ```

- BFS is complete for finite branching and optimal when every edge has equal cost

- Time and memory are approximately `O(b^d)`, where `b` is branching factor and `d` is shallowest goal depth

- Memory is often the limiting factor

# Depth-First Search

- DFS expands one branch deeply using a stack or recursion

    ```python
    def dfs(start, is_goal, neighbors):
        frontier = [(start, [start])]
        visited = set()

        while frontier:
            state, path = frontier.pop()

            if is_goal(state):
                return path

            if state in visited:
                continue

            visited.add(state)
            for neighbor in neighbors(state):
                frontier.append((neighbor, path + [neighbor]))

        return None
    ```

- DFS uses approximately `O(bm)` memory where `m` is maximum depth, but may spend a long time in poor branches

- DFS is not normally optimal and is not complete in infinite-depth spaces without controls

- Depth-Limited Search adds a maximum depth

- Iterative Deepening repeatedly runs depth-limited DFS with increasing limits. It combines BFS-like shallow-solution behavior with low DFS memory for uniform step costs

# Uniform-Cost Search and Dijkstra

- Uniform-Cost Search expands the node with smallest path cost `g(n)` using a priority queue

- Dijkstra's algorithm is the graph shortest-path form for nonnegative edge weights

    ```python
    import heapq
    import itertools


    def uniform_cost(start, is_goal, neighbors):
        counter = itertools.count()
        frontier = [(0, next(counter), start)]
        best_cost = {start: 0}
        parent = {start: None}

        while frontier:
            cost, _, state = heapq.heappop(frontier)

            if cost != best_cost[state]:
                continue

            if is_goal(state):
                return reconstruct(parent, state), cost

            for neighbor, step_cost in neighbors(state):
                new_cost = cost + step_cost
                if new_cost < best_cost.get(neighbor, float("inf")):
                    best_cost[neighbor] = new_cost
                    parent[neighbor] = state
                    heapq.heappush(frontier, (new_cost, next(counter), neighbor))

        return None
    ```

- Stale priority-queue entries may remain after a better path is found. Skip them when popped

- Negative edges invalidate Dijkstra/UCS assumptions. Bellman-Ford handles negative edges and detects reachable negative cycles

# Greedy Best-First Search

- Greedy search expands the node with smallest heuristic estimate `h(n)`

- It may reach a goal quickly with a strong heuristic but ignores the cost already paid

- It is not generally optimal and can be misled by heuristic valleys or loops

- A heuristic is domain information estimating remaining work

# A* Search

- A* ranks nodes by:

    `f(n) = g(n) + h(n)`

- `g(n)` protects the known cost from the start. `h(n)` estimates remaining cost

- An admissible heuristic never overestimates true remaining cost

- A consistent heuristic satisfies `h(n) <= cost(n, n') + h(n')`

- With appropriate graph-search handling and a consistent/admissible heuristic, A* is complete and optimal under normal positive-cost assumptions

    ```python
    def a_star(start, goal, neighbors, heuristic):
        counter = itertools.count()
        frontier = [(heuristic(start, goal), next(counter), start)]
        g_score = {start: 0}
        parent = {start: None}

        while frontier:
            _, _, state = heapq.heappop(frontier)

            if state == goal:
                return reconstruct(parent, state), g_score[state]

            for neighbor, cost in neighbors(state):
                candidate = g_score[state] + cost

                if candidate < g_score.get(neighbor, float("inf")):
                    parent[neighbor] = state
                    g_score[neighbor] = candidate
                    f_score = candidate + heuristic(neighbor, goal)
                    heapq.heappush(frontier, (f_score, next(counter), neighbor))

        return None
    ```

- A stronger admissible heuristic dominates a weaker one when it is always at least as large without overestimating

- If `h(n) = 0`, A* becomes Uniform-Cost Search

- Weighted A* uses `g(n) + w*h(n)` with `w > 1` to trade optimality guarantees for more aggressive goal direction

# Heuristic Design

- Derive an admissible heuristic by solving a relaxed version of the problem

- Manhattan distance fits four-direction grids without diagonal movement and with suitable costs

- Euclidean distance fits geometric straight-line lower bounds

- Pattern databases precompute exact solutions for abstracted subproblems

- Combine admissible heuristics with `max(h1, h2)` to retain admissibility and use the strongest estimate at each state

- Summing heuristics is only admissible when they measure disjoint cost components appropriately

- Heuristic error is the difference between estimate and true remaining cost

- Heuristic variance across successors measures how clearly the heuristic distinguishes branch promise, but high variance can indicate either useful signal or unstable noise

- Validate heuristics on known optimal small instances before depending on theoretical properties

# Beam and Width-Limited Search

- Beam search keeps only the best `k` candidates at each step or layer

- Small width saves memory and computation but may permanently discard the only successful or optimal branch

- Large width approaches broader best-first exploration

- Beam search is common in sequence decoding and large combinatorial spaces

- Diversity penalties or grouped beams reduce many candidates collapsing into near-duplicates

- Adaptive-width search changes width from observed confidence, heuristic separation, depth, remaining budget, or search progress

- Any adaptive rule should define minimum/maximum width and behavior when the signal is missing, tied, noisy, or misleading

# Confidence and Variance Guided Search

- Confidence-guided search estimates how trustworthy the current ranking is

- Possible signals:

    Gap between the best and second-best heuristic scores

    Normalized spread/variance among successor scores

    Agreement among multiple heuristics

    Historical success of similar rankings

    Search stagnation or improvement rate

- Large score separation may justify narrow exploitation if the heuristic is calibrated

- Similar scores may justify wider exploration because ranking provides little confidence

- Raw variance depends on score scale. Normalize by mean magnitude, range, standard deviation baseline, or a learned calibration

- A generic width controller is:

    ```python
    def adaptive_width(scores, min_width, max_width, confidence):
        uncertainty = 1.0 - confidence(scores)
        width = round(min_width + uncertainty * (max_width - min_width))
        return max(min_width, min(max_width, width))
    ```

- Define whether higher scores are better or worse before calculating gaps

- Deterministic tie-breaking is important for reproducible comparisons

- An adaptive algorithm can degrade when the heuristic is confidently wrong. Include fallback expansion, periodic diversification, restart, or budget reserve when completeness matters

# Local Search and Optimization

- Local search keeps one or a few current candidates rather than complete paths

- Hill climbing repeatedly moves to an improving neighbor and may stop at local optima or plateaus

- Simulated annealing sometimes accepts worse moves, with acceptance decreasing as temperature cools

- Tabu search remembers recent moves/states to reduce cycling

- Genetic algorithms use selection, crossover, and mutation over a population

- Local search is suitable when only the final configuration matters and path reconstruction is unnecessary

- Objective functions should separate feasibility penalties from optimization quality where possible

# Constraint Satisfaction

- A Constraint Satisfaction Problem contains variables, domains, and constraints

- Backtracking assigns one variable at a time and reverses when constraints fail

- Useful heuristics:

    MRV: Choose the variable with fewest legal values

    Degree heuristic: Choose the variable constraining the most unassigned variables

    Least-constraining value: Try the value eliminating the fewest neighbor options

- Forward checking removes values inconsistent with a new assignment

- Arc consistency propagates binary constraints more deeply

- Branch and bound prunes any partial solution whose optimistic bound cannot beat the best known complete solution

# Game Search

- Minimax chooses a move assuming an optimal opponent

- Alpha-beta pruning skips branches which cannot affect the minimax result

- Good move ordering greatly improves alpha-beta pruning

- Iterative deepening provides a usable best move under time limits

- Transposition tables cache positions reached through different move orders

- Monte Carlo Tree Search balances exploration and exploitation through repeated selection, expansion, simulation/evaluation, and backpropagation

# Reconstructing Paths

- Store only a parent/action reference per discovered state rather than copying full paths into every frontier entry when memory matters

    ```python
    def reconstruct(parent, goal):
        path = []
        state = goal

        while state is not None:
            path.append(state)
            state = parent[state]

        return list(reversed(path))
    ```

- For weighted algorithms, update the parent whenever a strictly better path replaces the prior best cost

# Benchmarking Search Algorithms

- A fair benchmark uses the same problem instances, state representation, successor generation, goal test, cost function, machine, timeout, and measurement method

- Compare against meaningful baselines such as DFS, BFS, UCS/Dijkstra, greedy best-first, A*, fixed-width beam, and random/restart methods where applicable

- Record:

    Success rate

    Runtime

    Nodes generated and expanded

    Peak frontier size and process memory

    Solution cost/quality

    Optimality gap

    Search depth

    Heuristic evaluations

    Width decisions over time

- Separate algorithm time from instance loading, result serialization, and visualization

- Use `time.perf_counter()` for elapsed time and repeat short runs enough to reduce timer noise

- Warm-up effects, caching, garbage collection, CPU frequency, and other processes may influence results

- Use multiple random seeds and report distribution/uncertainty, not only the best run

- Enforce equal time, node, or memory budgets when comparing anytime algorithms

- Validate returned solutions independently and calculate objective values with a shared verifier

- Avoid benchmark leakage: do not tune algorithm parameters on the final test instances

# Benchmark Framework Structure

    ```python
    from dataclasses import dataclass
    from time import perf_counter


    @dataclass
    class SearchResult:
        solved: bool
        solution: object | None
        cost: float | None
        runtime_seconds: float
        generated: int
        expanded: int
        peak_frontier: int


    def benchmark(algorithm, problem) -> SearchResult:
        start = perf_counter()
        result = algorithm(problem)
        elapsed = perf_counter() - start

        assert verify_solution(problem, result.solution)

        return SearchResult(
            solved=result.solved,
            solution=result.solution,
            cost=result.cost,
            runtime_seconds=elapsed,
            generated=result.generated,
            expanded=result.expanded,
            peak_frontier=result.peak_frontier,
        )
    ```

- Define one algorithm protocol so implementations receive identical problem interfaces and budgets

- Save raw per-instance results before aggregating them

- Include algorithm/version, parameters, seed, instance ID, environment, timestamp, and commit hash

- Use CSV/Parquet/JSON for raw data and generate plots/tables from saved results instead of rerunning invisibly

# Experimental Analysis

- Runtime alone does not explain why an algorithm succeeds

- Expansion count measures algorithmic work more independently from implementation speed

- Peak frontier approximates memory pressure

- Solution quality must be compared against known optimum, best-known solution, or a shared lower bound

- Performance profiles show how often algorithms solve within a factor of the best result

- Ablation studies remove one adaptive signal or rule to determine which component causes gains

- Sensitivity studies vary width limits, confidence thresholds, heuristic strength, noise, and budgets

- Stratify results by problem size, branching factor, heuristic error, constraint density, and solution depth

- A method which wins on average may still fail badly in a particular domain. Report successful and degrading domains

- Use paired comparisons because every algorithm should run the same instance set

- Practical significance matters in addition to statistical significance

# Common Mistakes

- Marking a state visited when generated in a weighted search and refusing a later cheaper path

- Comparing heap tuples containing unorderable states without a deterministic counter tie-breaker

- Using a heuristic with the wrong scale relative to path cost

- Claiming optimality without verifying admissibility/consistency and duplicate handling

- Calling one run a benchmark

- Reporting only solved instances and hiding timeouts/failures

- Tuning on the test set

- Using different termination budgets across algorithms

- Copying complete paths into every node and attributing memory growth only to the search strategy

- Measuring Python implementation overhead as if it were purely algorithmic behavior

- Treating heuristic confidence as truth without testing confidently wrong cases

# Quick Reference

- Shallow unweighted solution -> BFS

- Low memory and any solution -> DFS/iterative deepening

- Cheapest path with nonnegative costs -> UCS/Dijkstra

- Strong heuristic and optimal path -> A*

- Fast heuristic-directed attempt -> Greedy best-first

- Strict memory/branch budget -> Beam search

- Unknown useful width -> Adaptive-width search with bounded fallback

- Final configuration rather than path -> Local search

- Variable/domain constraints -> CSP backtracking and propagation

- Adversarial moves -> Minimax/alpha-beta or MCTS

- Fair algorithm claim -> Shared benchmark protocol, baselines, raw results, multiple seeds, and solution verifier
