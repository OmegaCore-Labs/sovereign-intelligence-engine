# Branching Timelines — Architecture Specification

## Overview

The Branching Timelines module simulates how timelines diverge at decision points, whether human choices, quantum events, or deterministic bifurcations. Each branch represents a distinct causal pathway with its own probability weight and subsequent evolution.

## Core Components

### 1. Timeline Branch Definition

```python
class TimelineBranch:
    """
    Representation of a single timeline branch.
    """
    
    def __init__(self, branch_id: str, parent_id: Optional[str] = None):
        self.id = branch_id
        self.parent_id = parent_id
        self.children = []
        self.probability = 1.0
        self.amplitude = 1.0  # Quantum amplitude if applicable
        self.state = {}  # Universe state at this branch point
        self.decision_point = None
        self.timestamp = 0.0
        
    def add_child(self, child_branch: 'TimelineBranch'):
        self.children.append(child_branch)
        
    def path_to_root(self) -> List[str]:
        """Return branch IDs from root to this branch."""
        path = [self.id]
        current = self
        while current.parent_id:
            path.insert(0, current.parent_id)
            # Find parent (simplified)
        return path
2. Decision Point Simulation
python
class DecisionPoint:
    """
    A point where timeline branches occur.
    """
    
    def __init__(self, description: str, timestamp: float):
        self.description = description
        self.timestamp = timestamp
        self.options = []
        self.weights = []
        self.state_before = {}
        
    def add_option(self, option_description: str, probability: float, 
                   state_delta: Dict):
        """
        Add a possible outcome at this decision point.
        """
        self.options.append({
            'description': option_description,
            'probability': probability,
            'state_delta': state_delta
        })
        self.weights.append(probability)
        
    def branch(self, parent_branch: TimelineBranch) -> List[TimelineBranch]:
        """
        Create child branches for each option.
        """
        children = []
        for i, option in enumerate(self.options):
            child_id = f"{parent_branch.id}.{i}"
            child = TimelineBranch(child_id, parent_branch.id)
            child.decision_point = self
            child.probability = option['probability'] * parent_branch.probability
            child.state = self._apply_delta(parent_branch.state, option['state_delta'])
            child.timestamp = self.timestamp
            parent_branch.add_child(child)
            children.append(child)
        return children
        
    def _apply_delta(self, state: Dict, delta: Dict) -> Dict:
        """Apply state changes for this branch."""
        new_state = state.copy()
        for key, value in delta.items():
            if callable(value):
                new_state[key] = value(new_state.get(key))
            else:
                new_state[key] = value
        return new_state
3. Timeline Tree Management
python
class TimelineTree:
    """
    Manages the complete tree of branching timelines.
    """
    
    def __init__(self, initial_state: Dict):
        self.root = TimelineBranch("root")
        self.root.state = initial_state
        self.root.timestamp = 0.0
        self.branches = {"root": self.root}
        self.current_branch_id = "root"
        self.branch_history = []
        
    def add_decision_point(self, decision: DecisionPoint, 
                           branch_id: Optional[str] = None) -> List[str]:
        """
        Apply a decision point to a branch, creating new branches.
        """
        if branch_id is None:
            branch_id = self.current_branch_id
            
        branch = self.branches[branch_id]
        new_branches = decision.branch(branch)
        
        for b in new_branches:
            self.branches[b.id] = b
            
        return [b.id for b in new_branches]
        
    def switch_to_branch(self, branch_id: str):
        """Move current position to specified branch."""
        if branch_id not in self.branches:
            raise ValueError(f"Branch {branch_id} not found")
        self.current_branch_id = branch_id
        self.branch_history.append(branch_id)
        
    def get_branch_path(self, branch_id: str) -> List[Dict]:
        """
        Get the sequence of decisions leading to a branch.
        """
        path = []
        current = self.branches[branch_id]
        
        while current.parent_id:
            if current.decision_point:
                path.insert(0, {
                    'branch_id': current.id,
                    'decision': current.decision_point.description,
                    'option': self._find_option(current),
                    'timestamp': current.timestamp
                })
            current = self.branches[current.parent_id]
            
        return path
        
    def _find_option(self, branch: TimelineBranch) -> str:
        """Find which option led to this branch."""
        if not branch.parent_id or not branch.decision_point:
            return "root"
        parent = self.branches[branch.parent_id]
        for i, child in enumerate(parent.children):
            if child.id == branch.id:
                return branch.decision_point.options[i]['description']
        return "unknown"
        
    def probability_of_branch(self, branch_id: str) -> float:
        """Calculate total probability of reaching a branch."""
        branch = self.branches[branch_id]
        return branch.probability
4. Timeline Analytics
python
class TimelineAnalytics:
    """
    Analyze properties of branching timeline structures.
    """
    
    def __init__(self, tree: TimelineTree):
        self.tree = tree
        
    def branch_statistics(self) -> Dict:
        """Calculate statistics about the timeline tree."""
        total_branches = len(self.tree.branches)
        max_depth = 0
        leaf_count = 0
        
        def traverse(node, depth):
            nonlocal max_depth, leaf_count
            max_depth = max(max_depth, depth)
            if not node.children:
                leaf_count += 1
            for child in node.children:
                traverse(child, depth + 1)
                
        traverse(self.tree.root, 0)
        
        return {
            'total_branches': total_branches,
            'max_depth': max_depth,
            'leaf_count': leaf_count,
            'branching_factor': total_branches / max_depth if max_depth > 0 else 0
        }
        
    def most_probable_path(self) -> List[str]:
        """Find the path with highest cumulative probability."""
        max_prob = 0
        best_path = []
        
        def traverse(node, path, prob):
            nonlocal max_prob, best_path
            current_path = path + [node.id]
            
            if not node.children:
                if prob > max_prob:
                    max_prob = prob
                    best_path = current_path
            else:
                for child in node.children:
                    traverse(child, current_path, prob * child.probability)
                    
        traverse(self.tree.root, [], 1.0)
        return best_path
        
    def divergence_point(self, branch1_id: str, branch2_id: str) -> Optional[str]:
        """Find the point where two branches diverged."""
        path1 = self.tree.get_branch_path(branch1_id)
        path2 = self.tree.get_branch_path(branch2_id)
        
        for i, (p1, p2) in enumerate(zip(path1, path2)):
            if p1['branch_id'] != p2['branch_id']:
                return p1['branch_id'] if i > 0 else path1[i-1]['branch_id']
                
        return None  # One is ancestor of the other
5. Timeline Visualization
python
class TimelineVisualizer:
    """
    Generate visual representations of timeline trees.
    """
    
    def __init__(self, tree: TimelineTree):
        self.tree = tree
        
    def ascii_tree(self, max_depth: Optional[int] = None) -> str:
        """
        Generate ASCII representation of timeline tree.
        """
        lines = []
        
        def traverse(node, prefix, is_last):
            # Current node
            line = prefix
            line += "└── " if is_last else "├── "
            prob_pct = node.probability * 100
            line += f"[{node.id}] ({prob_pct:.1f}%)"
            lines.append(line)
            
            # Prepare prefix for children
            child_prefix = prefix + ("    " if is_last else "│   ")
            
            # Process children
            for i, child in enumerate(node.children):
                is_last_child = (i == len(node.children) - 1)
                traverse(child, child_prefix, is_last_child)
                
        traverse(self.tree.root, "", True)
        return "\n".join(lines)
        
    def to_graphviz(self) -> str:
        """
        Generate Graphviz DOT format for visualization.
        """
        dot = ["digraph TimelineTree {"]
        dot.append("  rankdir=TB;")
        dot.append("  node [shape=box];")
        
        def add_node(node):
            prob_pct = node.probability * 100
            label = f"{node.id}\\n({prob_pct:.1f}%)"
            dot.append(f'  "{node.id}" [label="{label}"];')
            
            if node.parent_id:
                dot.append(f'  "{node.parent_id}" -> "{node.id}";')
                
            for child in node.children:
                add_node(child)
                
        add_node(self.tree.root)
        dot.append("}")
        return "\n".join(dot)
6. Integration Examples
python
# Example: Simulate historical decision point
timeline = TimelineTree(initial_state={
    'year': 1930,
    'technology_level': 0.3,
    'global_conflict_risk': 0.5
})

# Decision: Development of nuclear weapons
decision = DecisionPoint("Nuclear weapons development path", timestamp=1939)
decision.add_option(
    "Accelerated development (Manhattan Project style)",
    probability=0.4,
    state_delta={
        'technology_level': lambda x: x + 0.2,
        'global_conflict_risk': lambda x: x + 0.3,
        'nuclear_war_risk': 0.4
    }
)
decision.add_option(
    "Delayed development (international control)",
    probability=0.3,
    state_delta={
        'technology_level': lambda x: x + 0.05,
        'global_conflict_risk': lambda x: x - 0.1,
        'nuclear_war_risk': 0.1
    }
)
decision.add_option(
    "No development (treaty)",
    probability=0.3,
    state_delta={
        'technology_level': lambda x: x,
        'global_conflict_risk': lambda x: x - 0.2,
        'nuclear_war_risk': 0.0
    }
)

new_branches = timeline.add_decision_point(decision)

# Analyze outcomes
analytics = TimelineAnalytics(timeline)
stats = analytics.branch_statistics()
print(f"Created {stats['total_branches']} timeline branches")
print(f"Most probable path: {analytics.most_probable_path()}")
