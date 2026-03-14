# Higher-Order Thought (HOT) Theory — Architecture Specification

## Overview

Higher-Order Thought (HOT) theory proposes that a mental state becomes conscious when there is a higher-order thought about that state. This module implements computational models of HOT, enabling simulation of self-representation, metacognition, and the distinction between conscious and unconscious states.

## Core Architecture

### 1. Hierarchical State Representation

```python
class HierarchicalState:
    """
    Multi-level representation of mental states.
    
    Level 0: First-order (world-directed) states
    Level 1: Higher-order thoughts about level 0
    Level 2: Meta-cognitive thoughts about level 1
    """
    
    def __init__(self, max_depth: int = 3):
        self.max_depth = max_depth
        self.states = {i: [] for i in range(max_depth)}
        self.hot_relations = []  # (higher_idx, lower_idx) pairs
        
    def add_first_order_state(self, content: Any, confidence: float):
        """Add a world-directed mental state."""
        state_id = len(self.states[0])
        self.states[0].append({
            'id': state_id,
            'content': content,
            'confidence': confidence,
            'about_world': True
        })
        return state_id
        
    def add_hot(self, about_state_id: int, level: int, content: Any = None):
        """
        Add a higher-order thought about a lower-level state.
        """
        if level >= self.max_depth:
            raise ValueError(f"Max depth {self.max_depth} exceeded")
            
        # Verify target exists at lower level
        target_level = level - 1
        if about_state_id >= len(self.states[target_level]):
            raise ValueError(f"Target state {about_state_id} not found at level {target_level}")
            
        # Create HOT
        state_id = len(self.states[level])
        self.states[level].append({
            'id': state_id,
            'content': content or f"Thought about state {about_state_id}",
            'about': (target_level, about_state_id),
            'accuracy': None  # To be computed
        })
        
        self.hot_relations.append((level, state_id, target_level, about_state_id))
        return state_id
2. Consciousness Classification
python
class HOTConsciousnessClassifier:
    """
    Determine consciousness of states based on HOT presence.
    """
    
    def __init__(self, hierarchy: HierarchicalState):
        self.hierarchy = hierarchy
        
    def is_conscious(self, state_id: int, level: int = 0) -> bool:
        """
        A state is conscious iff there is a HOT about it.
        """
        # Check for any higher-order thought about this state
        for hot_level in range(level + 1, self.hierarchy.max_depth):
            for hot_state in self.hierarchy.states[hot_level]:
                if 'about' in hot_state:
                    target_level, target_id = hot_state['about']
                    if target_level == level and target_id == state_id:
                        return True
        return False
        
    def consciousness_report(self) -> Dict:
        """
        Generate report of conscious vs unconscious states.
        """
        report = {}
        for level in range(self.hierarchy.max_depth):
            level_states = []
            for state in self.hierarchy.states[level]:
                conscious = self.is_conscious(state['id'], level)
                level_states.append({
                    'id': state['id'],
                    'content': state['content'],
                    'conscious': conscious,
                    'hot_count': self._count_hots_about(state['id'], level)
                })
            report[f'level_{level}'] = level_states
        return report
        
    def _count_hots_about(self, state_id: int, level: int) -> int:
        """Count number of HOTs about this state."""
        count = 0
        for hot_level in range(level + 1, self.hierarchy.max_depth):
            for hot_state in self.hierarchy.states[hot_level]:
                if 'about' in hot_state:
                    target_level, target_id = hot_state['about']
                    if target_level == level and target_id == state_id:
                        count += 1
        return count
3. Metacognitive Monitoring
python
class MetacognitiveMonitor:
    """
    Monitor and evaluate accuracy of higher-order thoughts.
    """
    
    def __init__(self, hierarchy: HierarchicalState):
        self.hierarchy = hierarchy
        
    def evaluate_hot_accuracy(self) -> Dict:
        """
        Evaluate how accurate higher-order thoughts are.
        """
        results = []
        
        for hot_level in range(1, self.hierarchy.max_depth):
            for hot_state in self.hierarchy.states[hot_level]:
                if 'about' not in hot_state:
                    continue
                    
                target_level, target_id = hot_state['about']
                target = self.hierarchy.states[target_level][target_id]
                
                # HOT accuracy depends on whether its content matches
                # the actual content of the target state
                if 'content' in hot_state and 'content' in target:
                    accuracy = self._compute_content_match(
                        hot_state['content'],
                        target['content']
                    )
                else:
                    # If no explicit content, accuracy based on existence
                    accuracy = 1.0
                    
                hot_state['accuracy'] = accuracy
                results.append({
                    'hot_level': hot_level,
                    'hot_id': hot_state['id'],
                    'target_level': target_level,
                    'target_id': target_id,
                    'accuracy': accuracy
                })
                
        # Compute overall metacognitive sensitivity
        if results:
            avg_accuracy = np.mean([r['accuracy'] for r in results])
        else:
            avg_accuracy = 0.0
            
        return {
            'individual': results,
            'average_accuracy': avg_accuracy,
            'metacognitive_sensitivity': self._compute_d_prime(results)
        }
        
    def _compute_content_match(self, content1: Any, content2: Any) -> float:
        """Compute similarity between two content representations."""
        if isinstance(content1, str) and isinstance(content2, str):
            # Text similarity
            return 1.0 if content1 == content2 else 0.0
        elif isinstance(content1, np.ndarray) and isinstance(content2, np.ndarray):
            # Vector similarity
            return np.dot(content1, content2) / (np.linalg.norm(content1) * np.linalg.norm(content2) + 1e-8)
        else:
            # Default: exact match
            return 1.0 if content1 == content2 else 0.0
            
    def _compute_d_prime(self, results: List) -> float:
        """
        Compute d' (sensitivity) for metacognitive accuracy.
        """
        # Separate hits and false alarms
        hits = [r['accuracy'] for r in results if r['accuracy'] > 0.5]
        false_alarms = [1 - r['accuracy'] for r in results if r['accuracy'] <= 0.5]
        
        if not hits or not false_alarms:
            return 0.0
            
        hit_rate = np.mean(hits)
        fa_rate = np.mean(false_alarms)
        
        # Convert to z-scores (simplified)
        from scipy.stats import norm
        d_prime = norm.ppf(hit_rate) - norm.ppf(fa_rate)
        
        return d_prime
4. HOT Dynamics
python
class HOTDynamics:
    """
    Temporal dynamics of higher-order thought generation.
    """
    
    def __init__(self, hierarchy: HierarchicalState, 
                 hot_generation_rate: float = 0.1):
        self.hierarchy = hierarchy
        self.rate = hot_generation_rate
        self.time = 0
        
    def step(self, inputs: Dict[int, Any]):
        """
        Single timestep of HOT dynamics.
        
        1. Process first-order inputs
        2. Generate HOTs probabilistically
        3. Update HOTs based on feedback
        """
        # Step 1: Process first-order inputs
        for processor_id, content in inputs.items():
            state_id = self.hierarchy.add_first_order_state(
                content=content,
                confidence=np.random.random()
            )
            
        # Step 2: Generate HOTs
        self._generate_hots()
        
        # Step 3: Update based on feedback (learning)
        self._update_from_feedback()
        
        self.time += 1
        
    def _generate_hots(self):
        """
        Generate higher-order thoughts about existing states.
        """
        # For each existing state, possibly generate HOT
        for level in range(self.hierarchy.max_depth - 1):
            for state in self.hierarchy.states[level]:
                # Probability of HOT generation increases with:
                # - State confidence
                # - Novelty
                # - Relevance to current goals
                confidence = state.get('confidence', 0.5)
                hot_prob = self.rate * confidence
                
                if np.random.random() < hot_prob:
                    # Generate HOT at next level
                    self.hierarchy.add_hot(
                        about_state_id=state['id'],
                        level=level + 1,
                        content=f"HOT about {state['content']}"
                    )
                    
    def _update_from_feedback(self):
        """
        Update HOT generation based on feedback accuracy.
        """
        # Evaluate current HOTs
        monitor = MetacognitiveMonitor(self.hierarchy)
        accuracy_results = monitor.evaluate_hot_accuracy()
        
        # Adjust HOT generation rate based on accuracy
        if accuracy_results['average_accuracy'] < 0.7:
            # Reduce rate to avoid noisy HOTs
            self.rate *= 0.95
        elif accuracy_results['average_accuracy'] > 0.9:
            # Increase rate if accurate
            self.rate *= 1.05
            
        # Clamp to reasonable range
        self.rate = np.clip(self.rate, 0.01, 0.5)
5. Self-Representation
python
class SelfRepresentation:
    """
    Model of self as integrated set of higher-order thoughts.
    """
    
    def __init__(self, hierarchy: HierarchicalState):
        self.hierarchy = hierarchy
        self.self_model = {}  # Integrated self-representation
        self.narrative_self = []  # Autobiographical narrative
        
    def integrate_self(self):
        """
        Integrate HOTs into unified self-model.
        
        The self is the integrated set of higher-order thoughts
        about one's own mental states.
        """
        # Collect all HOTs (level >= 1)
        all_hots = []
        for level in range(1, self.hierarchy.max_depth):
            all_hots.extend(self.hierarchy.states[level])
            
        # Cluster HOTs by content similarity
        if all_hots:
            # Extract features from HOTs
            features = []
            for hot in all_hots:
                if 'content' in hot:
                    if isinstance(hot['content'], str):
                        # Simple bag-of-words
                        features.append(self._text_to_vector(hot['content']))
                    else:
                        features.append(np.array([0.0]))
                        
            if features:
                features = np.array(features)
                # Cluster into self-aspects
                from sklearn.cluster import KMeans
                n_clusters = min(5, len(features))
                if n_clusters > 1:
                    kmeans = KMeans(n_clusters=n_clusters)
                    clusters = kmeans.fit_predict(features)
                    
                    # Build self-model from clusters
                    for i in range(n_clusters):
                        cluster_hots = [all_hots[j] for j in range(len(all_hots)) if clusters[j] == i]
                        self.self_model[f'self_aspect_{i}'] = {
                            'hots': cluster_hots,
                            'center': kmeans.cluster_centers_[i],
                            'strength': len(cluster_hots) / len(all_hots)
                        }
                        
        return self.self_model
        
    def build_narrative(self, events: List[Dict]):
        """
        Build autobiographical narrative from events and HOTs.
        """
        narrative = []
        
        for event in events:
            # Find HOTs about this event
            relevant_hots = []
            for level in range(1, self.hierarchy.max_depth):
                for hot in self.hierarchy.states[level]:
                    if 'about' in hot:
                        target_level, target_id = hot['about']
                        if target_level == 0 and target_id == event.get('state_id'):
                            relevant_hots.append(hot)
                            
            # Create narrative element
            narrative_entry = {
                'event': event,
                'interpretation': relevant_hots,
                'self_relevance': len(relevant_hots) / (self.hierarchy.max_depth + 1)
            }
            narrative.append(narrative_entry)
            
        self.narrative_self = narrative
        return narrative
6. Clinical Applications
python
class HOTClinicalAssessment:
    """
    Assess disorders of consciousness using HOT framework.
    """
    
    def __init__(self, hierarchy: HierarchicalState):
        self.hierarchy = hierarchy
        
    def assess_consciousness_level(self) -> str:
        """
        Determine level of consciousness based on HOT presence.
        """
        # Count HOTs at each level
        hot_counts = [len(self.hierarchy.states[level]) for level in range(1, self.hierarchy.max_depth)]
        
        if hot_counts[0] == 0:  # No level 1 HOTs
            return "Unconscious (no higher-order thoughts)"
        elif hot_counts[1] == 0:  # Level 1 HOTs but no level 2
            return "Minimally conscious (first-order HOTs only)"
        elif hot_counts[2] == 0:  # Level 2 HOTs but no level 3
            return "Conscious with limited metacognition"
        else:
            return "Fully conscious (multi-level metacognition)"
            
    def detect_anosognosia(self) -> bool:
        """
        Detect lack of awareness of deficits (anosognosia).
        """
        monitor = MetacognitiveMonitor(self.hierarchy)
        accuracy = monitor.evaluate_hot_accuracy()
        
        # Look for systematic mismatch between HOT content and actual states
        mismatches = []
        for result in accuracy['individual']:
            if result['accuracy'] < 0.3:  # Significant mismatch
                mismatches.append(result)
                
        # If many mismatches and no awareness of them
        if len(mismatches) > len(accuracy['individual']) * 0.3:
            # Check for level 2 HOTs about the inaccurate level 1 HOTs
            level2_hots = self.hierarchy.states[2]
            for mismatch in mismatches:
                # Look for level 2 HOT about this inaccurate level 1 HOT
                has_meta = any(
                    hot.get('about') == (1, mismatch['hot_id'])
                    for hot in level2_hots
                )
                if not has_meta:
                    return True  # Found deficit without awareness
                    
        return False
Integration with Other Frameworks
python
class HOTWithIIT:
    """
    Integrate HOT with Integrated Information Theory.
    """
    
    def __init__(self, hot_system: HierarchicalState, iit_system):
        self.hot = hot_system
        self.iit = iit_system
        
    def compute_phi_of_hots(self) -> float:
        """
        Compute integrated information of HOT structure itself.
        """
        # Convert HOT hierarchy to IIT format
        hot_system = self._hots_to_system()
        
        # Compute Φ of HOT system
        phi, _, _ = system_phi(hot_system)
        
        return phi
        
    def _hots_to_system(self):
        """Convert HOT hierarchy to IIT-compatible dynamical system."""
        # Each HOT is an element
        n_elements = sum(len(self.hot.states[level]) for level in range(1, self.hot.max_depth))
        
        # Define transitions based on HOT relations
        def mechanism(states):
            # Complex mapping from HOT states to next states
            pass
            
        return IITSystem(n_elements, mechanism)
