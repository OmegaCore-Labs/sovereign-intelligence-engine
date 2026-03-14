# Timeline Navigation — Architecture Specification

## Overview

The Timeline Navigation module enables traversal between different timelines and universes, state transfer, and comparative analysis. It provides tools for exploring the multiverse and identifying points of convergence or divergence.

## Core Components

### 1. Timeline Navigator

```python
class TimelineNavigator:
    """
    Navigate between different timelines and universes.
    """
    
    def __init__(self):
        self.current_timeline_id = None
        self.current_universe_id = None
        self.navigation_history = []
        self.waypoints = {}  # Saved positions
        
    def jump_to_timeline(self, timeline_id: str, branch_point: Optional[str] = None):
        """
        Jump to a different timeline.
        """
        self.navigation_history.append({
            'from_timeline': self.current_timeline_id,
            'to_timeline': timeline_id,
            'branch_point': branch_point,
            'timestamp': time.time()
        })
        
        self.current_timeline_id = timeline_id
        
    def jump_to_universe(self, universe_id: str):
        """
        Jump to a different parallel universe.
        """
        self.navigation_history.append({
            'from_universe': self.current_universe_id,
            'to_universe': universe_id,
            'timestamp': time.time()
        })
        
        self.current_universe_id = universe_id
        
    def set_waypoint(self, name: str):
        """
        Save current position as waypoint.
        """
        self.waypoints[name] = {
            'timeline_id': self.current_timeline_id,
            'universe_id': self.current_universe_id,
            'timestamp': time.time()
        }
        
    return_to_waypoint(self, name: str):
        """
        Return to saved waypoint.
        """
        if name not in self.waypoints:
            raise ValueError(f"Waypoint {name} not found")
            
        wp = self.waypoints[name]
        self.jump_to_timeline(wp['timeline_id'])
        self.jump_to_universe(wp['universe_id'])
2. State Transfer
python
class StateTransfer:
    """
    Transfer state between timelines/universes.
    """
    
    def __init__(self):
        self.transfer_history = []
        self.compatibility_cache = {}
        
    def transfer_state(self, source: Dict, target: Dict, 
                        entities: List[str]) -> Dict:
        """
        Transfer specified entities from source to target.
        """
        # Check compatibility
        compatibility = self.check_compatibility(source, target)
        
        if compatibility < 0.5:
            print(f"Warning: Low compatibility ({compatibility:.2f})")
            
        transferred = {}
        for entity in entities:
            if entity in source['state']:
                # Apply transformation based on compatibility
                transformed = self._transform_state(
                    source['state'][entity],
                    compatibility
                )
                transferred[entity] = transformed
                
        # Record transfer
        self.transfer_history.append({
            'source_id': source.get('id'),
            'target_id': target.get('id'),
            'entities': entities,
            'compatibility': compatibility,
            'timestamp': time.time()
        })
        
        return transferred
        
    def check_compatibility(self, timeline1: Dict, timeline2: Dict) -> float:
        """
        Check how compatible two timelines are for state transfer.
        """
        # Create cache key
        key = (timeline1.get('id'), timeline2.get('id'))
        if key in self.compatibility_cache:
            return self.compatibility_cache[key]
            
        # Compare physical constants
        const_match = self._compare_constants(
            timeline1.get('constants', {}),
            timeline2.get('constants', {})
        )
        
        # Compare laws
        law_match = self._compare_laws(
            timeline1.get('laws', {}),
            timeline2.get('laws', {})
        )
        
        # Compare historical divergence
        hist_match = self._compare_history(
            timeline1.get('history', []),
            timeline2.get('history', [])
        )
        
        # Weighted average
        compatibility = (
            0.4 * const_match +
            0.3 * law_match +
            0.3 * hist_match
        )
        
        self.compatibility_cache[key] = compatibility
        return compatibility
        
    def _transform_state(self, state: Any, compatibility: float) -> Any:
        """
        Transform state based on timeline compatibility.
        """
        if compatibility > 0.9:
            # Almost identical - direct transfer
            return state
        elif compatibility > 0.7:
            # Minor adjustments needed
            return self._minor_adjustment(state)
        elif compatibility > 0.4:
            # Significant transformation
            return self._major_transformation(state)
        else:
            # Incompatible - return None or default
            return None
            
    def _compare_constants(self, const1: Dict, const2: Dict) -> float:
        """Compare physical constants between timelines."""
        if not const1 or not const2:
            return 0.5
            
        matches = 0
        total = 0
        
        all_consts = set(const1.keys()) | set(const2.keys())
        for const in all_consts:
            if const in const1 and const in const2:
                # Relative difference
                v1, v2 = const1[const], const2[const]
                if v1 != 0:
                    rel_diff = abs(v1 - v2) / abs(v1)
                    matches += max(0, 1 - rel_diff)
            total += 1
            
        return matches / total if total > 0 else 0.5
        
    def _compare_laws(self, laws1: Dict, laws2: Dict) -> float:
        """Compare physical laws between timelines."""
        if not laws1 or not laws2:
            return 0.5
            
        matches = 0
        total = 0
        
        all_domains = set(laws1.keys()) | set(laws2.keys())
        for domain in all_domains:
            if domain in laws1 and domain in laws2:
                if laws1[domain] == laws2[domain]:
                    matches += 1
            total += 1
            
        return matches / total if total > 0 else 0.5
        
    def _compare_history(self, hist1: List, hist2: List) -> float:
        """Compare historical timelines."""
        if not hist1 or not hist2:
            return 0.5
            
        # Find common events
        common = 0
        for e1 in hist1[:100]:  # Limit to recent history
            for e2 in hist2[:100]:
                if e1.get('id') == e2.get('id'):
                    common += 1
                    break
                    
        return common / min(len(hist1), len(hist2), 100)
3. Timeline Comparison
python
class TimelineComparator:
    """
    Compare multiple timelines side by side.
    """
    
    def __init__(self):
        self.comparison_results = {}
        
    def compare_timelines(self, timelines: List[Dict], 
                           metrics: List[str]) -> pd.DataFrame:
        """
        Compare timelines across specified metrics.
        """
        data = []
        
        for timeline in timelines:
            row = {'timeline_id': timeline.get('id')}
            
            for metric in metrics:
                if metric == 'population':
                    row[metric] = timeline.get('population', 0)
                elif metric == 'technology_level':
                    row[metric] = timeline.get('technology_level', 0)
                elif metric == 'events_count':
                    row[metric] = len(timeline.get('history', []))
                elif metric == 'divergence_point':
                    row[metric] = self._find_divergence(timeline, timelines[0])
                    
            data.append(row)
            
        df = pd.DataFrame(data).set_index('timeline_id')
        self.comparison_results[time.time()] = df
        return df
        
    def _find_divergence(self, timeline: Dict, reference: Dict) -> Optional[float]:
        """Find when timeline diverged from reference."""
        hist_t = timeline.get('history', [])
        hist_r = reference.get('history', [])
        
        for i, (e_t, e_r) in enumerate(zip(hist_t, hist_r)):
            if e_t.get('id') != e_r.get('id'):
                return e_t.get('timestamp', i)
                
        return None
        
    def highlight_differences(self, timeline1: Dict, timeline2: Dict) -> Dict:
        """
        Highlight key differences between two timelines.
        """
        differences = {
            'constants': {},
            'laws': {},
            'events': [],
            'statistics': {}
        }
        
        # Compare constants
        const1 = timeline1.get('constants', {})
        const2 = timeline2.get('constants', {})
        for const in set(const1.keys()) | set(const2.keys()):
            if const in const1 and const in const2:
                if const1[const] != const2[const]:
                    differences['constants'][const] = {
                        'timeline1': const1[const],
                        'timeline2': const2[const],
                        'ratio': const2[const] / const1[const] if const1[const] != 0 else float('inf')
                    }
                    
        # Compare laws
        laws1 = timeline1.get('laws', {})
        laws2 = timeline2.get('laws', {})
        for domain in set(laws1.keys()) | set(laws2.keys()):
            if laws1.get(domain) != laws2.get(domain):
                differences['laws'][domain] = {
                    'timeline1': laws1.get(domain),
                    'timeline2': laws2.get(domain)
                }
                
        # Find divergent events
        hist1 = timeline1.get('history', [])
        hist2 = timeline2.get('history', [])
        for i, (e1, e2) in enumerate(zip(hist1, hist2)):
            if e1.get('id') != e2.get('id'):
                differences['events'].append({
                    'position': i,
                    'timeline1_event': e1,
                    'timeline2_event': e2
                })
                break
                
        return differences
4. Convergence Detection
python
class ConvergenceDetector:
    """
    Detect points where timelines converge or synchronize.
    """
    
    def __init__(self):
        self.convergence_points = []
        
    def find_convergences(self, timelines: List[Dict], 
                           threshold: float = 0.8) -> List[Dict]:
        """
        Find points where multiple timelines converge.
        """
        convergences = []
        
        # Compare all pairs
        for i, t1 in enumerate(timelines):
            for j, t2 in enumerate(timelines[i+1:], i+1):
                conv = self._check_pair_convergence(t1, t2, threshold)
                if conv:
                    convergences.append({
                        'timeline1': t1.get('id'),
                        'timeline2': t2.get('id'),
                        'point': conv
                    })
                    
        return convergences
        
    def _check_pair_convergence(self, t1: Dict, t2: Dict, 
                                  threshold: float) -> Optional[Dict]:
        """Check if two timelines converge at any point."""
        hist1 = t1.get('history', [])
        hist2 = t2.get('history', [])
        
        # Find matching event sequences
        matches = []
        for i, e1 in enumerate(hist1):
            for j, e2 in enumerate(hist2):
                if self._events_match(e1, e2, threshold):
                    matches.append((i, j, e1))
                    
        # Find longest matching sequence
        if len(matches) > 1:
            # Check if matches form a continuous sequence
            sequence = [matches[0]]
            for k in range(1, len(matches)):
                if (matches[k][0] == matches[k-1][0] + 1 and
                    matches[k][1] == matches[k-1][1] + 1):
                    sequence.append(matches[k])
                    
            if len(sequence) > 3:  # At least 3 matching events
                return {
                    'start_event': sequence[0][2],
                    'end_event': sequence[-1][2],
                    'length': len(sequence),
                    'timeline1_indices': [m[0] for m in sequence],
                    'timeline2_indices': [m[1] for m in sequence]
                }
                
        return None
        
    def _events_match(self, e1: Dict, e2: Dict, threshold: float) -> bool:
        """Check if two events are similar enough to count as match."""
        if e1.get('id') == e2.get('id'):
            return True
            
        # Check description similarity
        desc1 = e1.get('description', '')
        desc2 = e2.get('description', '')
        
        # Simple word overlap
        words1 = set(desc1.lower().split())
        words2 = set(desc2.lower().split())
        
        if words1 and words2:
            overlap = len(words1 & words2) / len(words1 | words2)
            return overlap > threshold
            
        return False
5. Multiverse Map
python
class MultiverseMap:
    """
    Create and manage a map of the multiverse.
    """
    
    def __init__(self):
        self.timelines = {}
        self.universes = {}
        self.connections = []  # Links between timelines/universes
        
    def add_timeline(self, timeline_id: str, properties: Dict):
        """Add a timeline to the map."""
        self.timelines[timeline_id] = {
            'id': timeline_id,
            'properties': properties,
            'connections': [],
            'position': self._assign_position()
        }
        
    def add_universe(self, universe_id: str, properties: Dict):
        """Add a parallel universe to the map."""
        self.universes[universe_id] = {
            'id': universe_id,
            'properties': properties,
            'connections': [],
            'position': self._assign_position()
        }
        
    def add_connection(self, source_id: str, target_id: str, 
                        strength: float = 1.0, type: str = 'unknown'):
        """Add a connection between two points in the multiverse."""
        self.connections.append({
            'source': source_id,
            'target': target_id,
            'strength': strength,
            'type': type
        })
        
        # Add to respective connection lists
        if source_id in self.timelines:
            self.timelines[source_id]['connections'].append(target_id)
        elif source_id in self.universes:
            self.universes[source_id]['connections'].append(target_id)
            
    def _assign_position(self) -> Tuple[float, float, float]:
        """Assign a 3D position for visualization."""
        # Simple random positioning
        return (
            np.random.uniform(-10, 10),
            np.random.uniform(-10, 10),
            np.random.uniform(-10, 10)
        )
        
    def to_graphviz(self) -> str:
        """Generate Graphviz DOT format for visualization."""
        dot = ["digraph MultiverseMap {"]
        dot.append("  rankdir=LR;")
        
        # Add timelines
        for tid, tdata in self.timelines.items():
            label = f"{tid}\\n{len(tdata['connections'])} connections"
            dot.append(f'  "{tid}" [shape=box, label="{label}"];')
            
        # Add universes
        for uid, udata in self.universes.items():
            label = f"{uid}\\n{len(udata['connections'])} connections"
            dot.append(f'  "{uid}" [shape=ellipse, label="{label}"];')
            
        # Add connections
        for conn in self.connections:
            strength_str = f" [label=\"{conn['strength']:.2f}\"]" if conn['strength'] != 1.0 else ""
            dot.append(f'  "{conn["source"]}" -> "{conn["target"]}"{strength_str};')
            
        dot.append("}")
        return "\n".join(dot)
6. Integration Examples
python
# Example: Navigate and compare timelines
from aris.architecture.multiverse import (
    TimelineNavigator,
    StateTransfer,
    TimelineComparator,
    MultiverseMap
)

# Initialize
navigator = TimelineNavigator()
transfer = StateTransfer()
comparator = TimelineComparator()
map_viz = MultiverseMap()

# Set current position
navigator.current_timeline_id = "timeline_earth_prime"
navigator.current_universe_id = "universe_alpha"

# Save waypoint
navigator.set_waypoint("home")

# Jump to alternate timeline
navigator.jump_to_timeline("timeline_napoleon_wins")

# Check compatibility
source = {'id': 'timeline_earth_prime', 'state': {'population': 8e9}}
target = {'id': 'timeline_napoleon_wins', 'state': {}}

compat = transfer.check_compatibility(source, target)
print(f"Timeline compatibility: {compat:.2f}")

# Transfer some entities
transferred = transfer.transfer_state(
    source=source,
    target=target,
    entities=['population', 'technology']
)

print(f"Transferred: {transferred}")

# Compare timelines
timelines_data = [
    {'id': 'prime', 'history': [{'id': 'event1'}, {'id': 'event2'}]},
    {'id': 'alt1', 'history': [{'id': 'event1'}, {'id': 'event3'}]},
    {'id': 'alt2', 'history': [{'id': 'event4'}, {'id': 'event5'}]}
]

comparison = comparator.compare_timelines(
    timelines=timelines_data,
    metrics=['events_count', 'divergence_point']
)

print("\nTimeline comparison:")
print(comparison)

# Add to multiverse map
for timeline in timelines_data:
    map_viz.add_timeline(timeline['id'], timeline)

map_viz.add_connection('prime', 'alt1', strength=0.7)
map_viz.add_connection('prime', 'alt2', strength=0.3)

# Generate visualization
print("\nMultiverse map (Graphviz):")
print(map_viz.to_graphviz())
