# Counterfactual Engine — Architecture Specification

## Overview

The Counterfactual Engine simulates "what-if" scenarios across historical events, personal decisions, and scientific possibilities. It generates plausible alternative realities by modifying key events and propagating consequences through causal networks.

## Core Components

### 1. Counterfactual Definition

```python
class Counterfactual:
    """
    Definition of a counterfactual scenario.
    """
    
    def __init__(self, name: str):
        self.name = name
        self.modified_events = []  # Events that are changed
        self.inserted_events = []  # New events added
        self.removed_events = []   # Events removed
        self.changed_parameters = {}  # Parameter modifications
        self.start_time = None
        self.end_time = None
        self.plausibility = 1.0  # 0-1 scale
        self.description = ""
        
    def modify_event(self, event_id: str, changes: Dict):
        """Modify an existing event."""
        self.modified_events.append({
            'event_id': event_id,
            'changes': changes
        })
        
    def insert_event(self, event: Dict):
        """Insert a new event."""
        self.inserted_events.append(event)
        
    def remove_event(self, event_id: str):
        """Remove an event."""
        self.removed_events.append(event_id)
        
    def set_parameter(self, name: str, value: Any):
        """Set a global parameter change."""
        self.changed_parameters[name] = value
2. Event Network
python
class EventNode:
    """
    A node in the causal event network.
    """
    
    def __init__(self, event_id: str, description: str, timestamp: float):
        self.id = event_id
        self.description = description
        self.timestamp = timestamp
        self.parents = []  # Causal parents
        self.children = []  # Causal children
        self.parameters = {}
        self.probability = 1.0
        self.counterfactual_versions = {}  # Alternative versions
        
    def add_parent(self, parent_id: str, strength: float = 1.0):
        """Add a causal parent."""
        self.parents.append({
            'id': parent_id,
            'strength': strength
        })
        
    def add_child(self, child_id: str, strength: float = 1.0):
        """Add a causal child."""
        self.children.append({
            'id': child_id,
            'strength': strength
        })
3. Causal Network
python
class CausalNetwork:
    """
    Network of causally connected events.
    """
    
    def __init__(self):
        self.events = {}  # id -> EventNode
        self.timeline = []  # Events in chronological order
        
    def add_event(self, event: EventNode):
        """Add an event to the network."""
        self.events[event.id] = event
        self._update_timeline()
        
    def _update_timeline(self):
        """Update chronological ordering."""
        self.timeline = sorted(
            self.events.values(),
            key=lambda e: e.timestamp
        )
        
    def get_causal_chain(self, event_id: str, direction: str = 'both') -> List[str]:
        """
        Get causal chain for an event.
        """
        chain = []
        visited = set()
        
        def traverse(node_id, dir):
            if node_id in visited:
                return
            visited.add(node_id)
            chain.append(node_id)
            
            node = self.events[node_id]
            if dir in ['parents', 'both']:
                for parent in node.parents:
                    traverse(parent['id'], dir)
            if dir in ['children', 'both']:
                for child in node.children:
                    traverse(child['id'], dir)
                    
        traverse(event_id, direction)
        return chain
        
    def propagate_change(self, event_id: str, changes: Dict) -> Dict:
        """
        Propagate changes through causal network.
        """
        affected = {}
        
        def propagate(node_id, change_strength):
            node = self.events[node_id]
            
            # Record effect
            affected[node_id] = {
                'change_strength': change_strength,
                'node': node
            }
            
            # Propagate to children with衰减
            for child in node.children:
                new_strength = change_strength * child['strength']
                propagate(child['id'], new_strength)
                
        propagate(event_id, 1.0)
        return affected
4. Counterfactual Engine
python
class CounterfactualEngine:
    """
    Main engine for generating and analyzing counterfactuals.
    """
    
    def __init__(self, causal_network: CausalNetwork):
        self.network = causal_network
        self.counterfactuals = {}
        self.results_cache = {}
        
    def generate_counterfactual(self, cf: Counterfactual) -> Dict:
        """
        Generate a counterfactual reality.
        """
        # Create modified network
        modified_network = self._apply_modifications(cf)
        
        # Propagate changes
        all_changes = {}
        for modified in cf.modified_events:
            changes = self._propagate_modification(
                modified['event_id'],
                modified['changes']
            )
            all_changes.update(changes)
            
        # Calculate divergence
        divergence = self._calculate_divergence(modified_network, all_changes)
        
        # Build result
        result = {
            'counterfactual': cf.name,
            'plausibility': cf.plausibility,
            'modified_network': modified_network,
            'affected_events': all_changes,
            'divergence_score': divergence,
            'new_timeline': self._build_timeline(modified_network)
        }
        
        self.counterfactuals[cf.name] = result
        return result
        
    def _apply_modifications(self, cf: Counterfactual) -> CausalNetwork:
        """Apply counterfactual modifications to network."""
        # Deep copy network
        import copy
        new_network = copy.deepcopy(self.network)
        
        # Apply modifications
        for mod in cf.modified_events:
            if mod['event_id'] in new_network.events:
                event = new_network.events[mod['event_id']]
                for key, value in mod['changes'].items():
                    event.parameters[key] = value
                    
        # Insert new events
        for event_data in cf.inserted_events:
            new_event = EventNode(
                event_data['id'],
                event_data['description'],
                event_data['timestamp']
            )
            new_event.parameters = event_data.get('parameters', {})
            new_network.add_event(new_event)
            
        # Remove events
        for event_id in cf.removed_events:
            if event_id in new_network.events:
                del new_network.events[event_id]
                
        return new_network
        
    def _propagate_modification(self, event_id: str, changes: Dict) -> Dict:
        """Propagate changes through network."""
        if event_id not in self.network.events:
            return {}
            
        return self.network.propagate_change(event_id, changes)
        
    def _calculate_divergence(self, modified_network: CausalNetwork,
                               changes: Dict) -> float:
        """Calculate how much the timeline diverges."""
        # Simple divergence metric
        if not changes:
            return 0.0
            
        # Average change strength weighted by causal impact
        total_strength = sum(
            info['change_strength'] for info in changes.values()
        )
        return total_strength / len(changes)
        
    def _build_timeline(self, network: CausalNetwork) -> List[Dict]:
        """Build timeline from modified network."""
        timeline = []
        for event in network.timeline:
            timeline.append({
                'id': event.id,
                'description': event.description,
                'timestamp': event.timestamp,
                'parameters': event.parameters
            })
        return timeline
        
    def compare_counterfactuals(self, cf_names: List[str]) -> pd.DataFrame:
        """
        Compare multiple counterfactual scenarios.
        """
        data = []
        for name in cf_names:
            if name in self.counterfactuals:
                cf = self.counterfactuals[name]
                data.append({
                    'name': name,
                    'plausibility': cf['plausibility'],
                    'divergence': cf['divergence_score'],
                    'affected_events': len(cf['affected_events'])
                })
                
        return pd.DataFrame(data)
5. Historical Counterfactuals
python
class HistoricalCounterfactuals:
    """
    Predefined historical counterfactual scenarios.
    """
    
    @staticmethod
    def archduke_assassination_fails() -> Counterfactual:
        """What if Archduke Franz Ferdinand survived?"""
        cf = Counterfactual("Archduke Survives")
        cf.description = "Archduke Franz Ferdinand's driver takes correct turn, avoiding assassination"
        cf.plausibility = 0.3
        
        cf.modify_event("assassination_attempt_1914", {
            'success': False,
            'casualties': 0
        })
        
        # Remove WWI trigger
        cf.removed_events.append("wwi_declaration")
        
        # Add alternative events
        cf.insert_event({
            'id': "austrian_diplomacy",
            'description': "Austria-Hungary pursues diplomatic solution",
            'timestamp': 1914.5,
            'parameters': {'outcome': 'negotiated_settlement'}
        })
        
        return cf
        
    @staticmethod
    def napoleon_wins_waterloo() -> Counterfactual:
        """What if Napoleon won at Waterloo?"""
        cf = Counterfactual("Napoleon Victorious")
        cf.description = "Napoleon defeats Wellington at Waterloo"
        cf.plausibility = 0.4
        
        cf.modify_event("battle_of_waterloo", {
            'outcome': 'french_victory',
            'napoleon_casualties': 15000,
            'allied_casualties': 35000
        })
        
        # Modify subsequent events
        cf.modify_event("congress_of_vienna", {
            'napoleon_present': True,
            'outcome': 'french_influence_maintained'
        })
        
        return cf
        
    @staticmethod
    def no_internet() -> Counterfactual:
        """What if the internet was never invented?"""
        cf = Counterfactual("No Internet")
        cf.description = "ARPANET project fails, internet never develops"
        cf.plausibility = 0.1
        
        cf.modify_event("arpanet_creation", {
            'success': False,
            'funding_cut': True
        })
        
        # Remove all internet-dependent events
        for year in range(1980, 2024):
            cf.removed_events.append(f"www_creation_{year}")
            cf.removed_events.append(f"social_media_{year}")
            cf.removed_events.append(f"ecommerce_{year}")
            
        return cf
6. Personal Counterfactuals
python
class PersonalCounterfactuals:
    """
    Personal decision counterfactuals.
    """
    
    def __init__(self, life_events: List[Dict]):
        self.life_events = life_events
        self.network = self._build_network()
        
    def _build_network(self) -> CausalNetwork:
        """Build causal network from life events."""
        network = CausalNetwork()
        
        for event in self.life_events:
            node = EventNode(
                event['id'],
                event['description'],
                event['timestamp']
            )
            node.parameters = event.get('parameters', {})
            network.add_event(node)
            
        # Build causal links (simplified - timestamp based)
        for i, event in enumerate(self.life_events):
            for j, other in enumerate(self.life_events):
                if i < j and other['timestamp'] - event['timestamp'] < 5:  # Within 5 years
                    network.events[event['id']].add_child(other['id'], strength=0.5)
                    
        return network
        
    def what_if(self, decision_point: str, alternative: str) -> Counterfactual:
        """Generate counterfactual for a personal decision."""
        cf = Counterfactual(f"What if {alternative}")
        
        cf.modify_event(decision_point, {
            'decision': alternative,
            'outcome': f"Chose {alternative} instead"
        })
        
        return cf
        
    def career_paths(self, current_career: str) -> List[Counterfactual]:
        """Generate alternative career paths."""
        alternatives = [
            "tech_entrepreneur",
            "artist",
            "scientist",
            "teacher",
            "doctor",
            "writer"
        ]
        
        counterfactuals = []
        for alt in alternatives:
            if alt != current_career:
                cf = Counterfactual(f"Career: {alt}")
                cf.modify_event("career_choice", {'career': alt})
                counterfactuals.append(cf)
                
        return counterfactuals
7. Integration Examples
python
# Example: Generate and analyze counterfactuals
from aris.architecture.multiverse import (
    CausalNetwork,
    CounterfactualEngine,
    HistoricalCounterfactuals
)

# Build historical event network
network = CausalNetwork()

# Add historical events
events = [
    EventNode("wwi_start", "World War I begins", 1914.5),
    EventNode("russian_revolution", "Russian Revolution", 1917.0),
    EventNode("treaty_versailles", "Treaty of Versailles", 1919.0),
    EventNode("great_depression", "Great Depression", 1929.0),
    EventNode("wwii_start", "World War II begins", 1939.0),
]

for event in events:
    network.add_event(event)

# Build causal links
network.events["wwi_start"].add_child("russian_revolution", 0.8)
network.events["wwi_start"].add_child("treaty_versailles", 1.0)
network.events["treaty_versailles"].add_child("great_depression", 0.6)
network.events["great_depression"].add_child("wwii_start", 0.7)

# Initialize counterfactual engine
engine = CounterfactualEngine(network)

# Generate historical counterfactuals
cf1 = HistoricalCounterfactuals.archduke_assassination_fails()
cf2 = HistoricalCounterfactuals.napoleon_wins_waterloo()

result1 = engine.generate_counterfactual(cf1)
result2 = engine.generate_counterfactual(cf2)

# Compare counterfactuals
comparison = engine.compare_counterfactuals(["Archduke Survives", "Napoleon Victorious"])
print(comparison)

# Visualize affected events
affected = result1['affected_events']
print(f"Events affected by Archduke surviving: {len(affected)}")
for event_id, info in list(affected.items())[:5]:
    print(f"  {event_id}: change strength {info['change_strength']:.2f}")
