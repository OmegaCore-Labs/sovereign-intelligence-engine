# Altered States of Consciousness — Architecture Specification

## Overview

Altered States module simulates modified modes of consciousness including meditation, psychedelic experiences, dreaming, hypnosis, and pathological states. This framework enables exploration of how consciousness transforms under different conditions and provides tools for analyzing state transitions.

## Core Architecture

### 1. State Space Model

```python
class AlteredStateSpace:
    """
    Multi-dimensional space of consciousness states.
    
    Dimensions capture key aspects of conscious experience:
    - Arousal: Wakefulness-sleep continuum
    - Awareness: Meta-cognitive access
    - Sensory gating: Filtering of input
    - Self-boundary: Ego dissolution
    - Time perception: Temporal flow alteration
    - Affect: Emotional valence
    - Coherence: Integration of experience
    """
    
    def __init__(self):
        self.dimensions = {
            'arousal': 1.0,           # 0=deep sleep, 1=hyper-aroused
            'awareness': 1.0,          # 0=unaware, 1=hyper-aware
            'sensory_gating': 0.5,      # 0=open, 1=filtered
            'self_boundary': 1.0,       # 0=dissolved, 1=rigid
            'time_perception': 1.0,     # 0=timeless, 1=normal, 2=accelerated
            'affect': 0.0,              # -1=negative, 0=neutral, 1=positive
            'coherence': 1.0            # 0=fragmented, 1=integrated
        }
        
        self.state_history = []
        self.current_state_name = "waking"
        
    def set_state(self, state_name: str, parameters: Dict[str, float]):
        """
        Set to a specific altered state.
        """
        for dim, value in parameters.items():
            if dim in self.dimensions:
                self.dimensions[dim] = np.clip(value, 0, 2)
                
        self.current_state_name = state_name
        self.state_history.append({
            'time': len(self.state_history),
            'state': state_name,
            'dimensions': self.dimensions.copy()
        })
        
    def transition_to(self, target_state: str, duration: float = 1.0):
        """
        Smooth transition to target state over duration.
        """
        # Get target parameters
        target_params = STATE_LIBRARY[target_state]
        
        # Generate trajectory
        trajectory = []
        steps = int(duration * 100)  # 10ms steps
        
        for t in range(steps):
            alpha = t / steps
            interpolated = {}
            
            for dim in self.dimensions:
                current = self.dimensions[dim]
                target = target_params.get(dim, current)
                interpolated[dim] = (1 - alpha) * current + alpha * target
                
            trajectory.append(interpolated)
            
        return trajectory
2. State Library
python
STATE_LIBRARY = {
    # Normal states
    "waking": {
        'arousal': 1.0,
        'awareness': 1.0,
        'sensory_gating': 0.5,
        'self_boundary': 1.0,
        'time_perception': 1.0,
        'affect': 0.0,
        'coherence': 1.0
    },
    
    "deep_sleep": {
        'arousal': 0.1,
        'awareness': 0.0,
        'sensory_gating': 1.0,
        'self_boundary': 0.0,
        'time_perception': 0.0,
        'affect': 0.0,
        'coherence': 0.1
    },
    
    "rem_sleep": {
        'arousal': 0.6,
        'awareness': 0.3,
        'sensory_gating': 0.8,
        'self_boundary': 0.3,
        'time_perception': 0.5,
        'affect': 0.5,
        'coherence': 0.4
    },
    
    # Meditative states
    "focused_attention": {
        'arousal': 0.8,
        'awareness': 1.2,
        'sensory_gating': 0.8,
        'self_boundary': 1.0,
        'time_perception': 0.8,
        'affect': 0.3,
        'coherence': 1.2
    },
    
    "open_monitoring": {
        'arousal': 0.7,
        'awareness': 1.3,
        'sensory_gating': 0.2,
        'self_boundary': 0.5,
        'time_perception': 0.6,
        'affect': 0.4,
        'coherence': 1.1
    },
    
    "non_dual": {
        'arousal': 0.6,
        'awareness': 1.5,
        'sensory_gating': 0.1,
        'self_boundary': 0.0,
        'time_perception': 0.2,
        'affect': 0.8,
        'coherence': 1.5
    },
    
    # Psychedelic states
    "psychedelic_low": {
        'arousal': 1.1,
        'awareness': 1.3,
        'sensory_gating': 0.3,
        'self_boundary': 0.6,
        'time_perception': 1.3,
        'affect': 0.7,
        'coherence': 0.8
    },
    
    "psychedelic_high": {
        'arousal': 1.2,
        'awareness': 1.5,
        'sensory_gating': 0.1,
        'self_boundary': 0.2,
        'time_perception': 1.8,
        'affect': 0.9,
        'coherence': 0.4
    },
    
    "psychedelic_ego_dissolution": {
        'arousal': 1.1,
        'awareness': 1.4,
        'sensory_gating': 0.1,
        'self_boundary': 0.0,
        'time_perception': 0.3,
        'affect': 1.0,
        'coherence': 0.3
    },
    
    # Hypnotic states
    "hypnosis_light": {
        'arousal': 0.6,
        'awareness': 0.8,
        'sensory_gating': 0.7,
        'self_boundary': 0.8,
        'time_perception': 0.7,
        'affect': 0.2,
        'coherence': 0.9
    },
    
    "hypnosis_deep": {
        'arousal': 0.4,
        'awareness': 0.5,
        'sensory_gating': 0.9,
        'self_boundary': 0.4,
        'time_perception': 0.4,
        'affect': 0.3,
        'coherence': 0.7
    },
    
    # Pathological states
    "schizophrenia_acute": {
        'arousal': 1.3,
        'awareness': 0.7,
        'sensory_gating': 0.1,
        'self_boundary': 0.3,
        'time_perception': 1.4,
        'affect': -0.3,
        'coherence': 0.2
    },
    
    "depression_severe": {
        'arousal': 0.3,
        'awareness': 0.4,
        'sensory_gating': 0.6,
        'self_boundary': 1.2,
        'time_perception': 0.4,
        'affect': -0.9,
        'coherence': 0.5
    },
    
    "mania": {
        'arousal': 1.8,
        'awareness': 0.9,
        'sensory_gating': 0.3,
        'self_boundary': 1.3,
        'time_perception': 1.9,
        'affect': 0.9,
        'coherence': 0.4
    }
}
3. Meditation Simulation
python
class MeditationSimulator:
    """
    Simulate meditation practice and its effects on consciousness.
    """
    
    def __init__(self, state_space: AlteredStateSpace):
        self.state = state_space
        self.practice_history = []
        self.expertise_level = 0.0  # 0=beginner, 1=expert
        
    def practice_session(self, technique: str, duration_minutes: float = 20):
        """
        Simulate a meditation session.
        """
        # Initialize at waking state
        self.state.set_state("waking", STATE_LIBRARY["waking"])
        
        trajectory = []
        
        for minute in range(int(duration_minutes)):
            # Progress through session
            progress = minute / duration_minutes
            
            if technique == "focused_attention":
                # FA: Increasing focus, then settling
                if progress < 0.3:
                    # Effortful focusing
                    target = "focused_attention"
                else:
                    # Stable focus
                    target = "open_monitoring"
                    
            elif technique == "open_monitoring":
                # OM: Gradual opening
                target = "open_monitoring"
                
            elif technique == "loving_kindness":
                # Metta: Increasing positive affect
                state = self.state.dimensions.copy()
                state['affect'] = min(1.0, progress * 1.5)
                target_params = state
                
            # Apply expertise modifier
            expertise_mod = 1.0 + self.expertise_level
            target_params = STATE_LIBRARY.get(target, STATE_LIBRARY["waking"]).copy()
            for dim in target_params:
                if dim in ['awareness', 'coherence']:
                    target_params[dim] *= expertise_mod
                    
            # Update state
            self.state.set_state(target, target_params)
            trajectory.append(self.state.dimensions.copy())
            
        # After-session effects
        self.state.set_state("waking", STATE_LIBRARY["waking"])
        
        # Update expertise
        self.expertise_level = min(1.0, self.expertise_level + 0.01)
        
        return trajectory
4. Psychedelic Experience Simulation
python
class PsychedelicSimulator:
    """
    Simulate psychedelic states and their phenomenological effects.
    """
    
    def __init__(self, state_space: AlteredStateSpace,
                 qualia_space=None):
        self.state = state_space
        self.qualia = qualia_space
        self.compound_database = {}
        
    def register_compound(self, name: str, effects: Dict):
        """
        Register psychedelic compound with its effect profile.
        """
        self.compound_database[name] = effects
        
    def experience(self, compound: str, dose: float = 1.0,
                   set_setting: Dict = None):
        """
        Simulate psychedelic experience.
        
        Includes:
        - Come-up
        - Peak
        - Plateau
        - Come-down
        - Afterglow
        """
        if compound not in self.compound_database:
            raise ValueError(f"Unknown compound: {compound}")
            
        effects = self.compound_database[compound]
        
        # Scale effects by dose
        for dim in effects:
            effects[dim] *= dose
            
        # Time course (minutes)
        come_up = 30
        peak = 120
        plateau = 60
        come_down = 90
        
        total_duration = come_up + peak + plateau + come_down
        experience_trajectory = []
        
        for minute in range(total_duration):
            if minute < come_up:
                # Come-up: linear increase
                progress = minute / come_up
                intensity = progress
                
            elif minute < come_up + peak:
                # Peak: maximum intensity
                intensity = 1.0
                
            elif minute < come_up + peak + plateau:
                # Plateau: sustained
                intensity = 0.9
                
            else:
                # Come-down: linear decrease
                progress = (minute - (come_up + peak + plateau)) / come_down
                intensity = 1.0 - progress
                
            # Apply current intensity to effects
            current_state = {}
            for dim, base_effect in effects.items():
                if dim in self.state.dimensions:
                    # Add psychedelic effect to baseline
                    baseline = STATE_LIBRARY["waking"][dim]
                    current_state[dim] = baseline + intensity * base_effect
                    
            self.state.set_state(f"psychedelic_{minute}", current_state)
            experience_trajectory.append(self.state.dimensions.copy())
            
        return experience_trajectory
        
    def visual_effects(self, intensity: float) -> Dict:
        """
        Generate visual phenomenology.
        """
        if self.qualia is None:
            return None
            
        effects = {
            'geometric_patterns': intensity * np.random.random((10, 10)),
            'color_enhancement': 1.0 + intensity,
            'tracers': intensity > 0.5,
            'fractal_imagery': intensity * 0.8,
            'synesthesia': intensity > 0.7
        }
        
        return effects
5. Dream Simulation
python
class DreamSimulator:
    """
    Simulate dream states and dream content.
    """
    
    def __init__(self, state_space: AlteredStateSpace,
                 memory_system=None):
        self.state = state_space
        self.memory = memory_system
        self.dream_content = []
        
    def enter_rem(self, duration_minutes: float = 20):
        """
        Simulate REM sleep and dreaming.
        """
        # Set state to REM
        self.state.set_state("rem_sleep", STATE_LIBRARY["rem_sleep"])
        
        # Generate dream content
        dream_cycles = int(duration_minutes / 5)  # ~5min per dream segment
        
        for cycle in range(dream_cycles):
            dream_segment = self._generate_dream_segment(cycle)
            self.dream_content.append(dream_segment)
            
            # Dream dynamics
            self._dream_narrative_progression(dream_segment)
            
        return self.dream_content
        
    def _generate_dream_segment(self, cycle: int) -> Dict:
        """
        Generate a segment of dream content.
        """
        # Sources of dream content
        if self.memory:
            # Memory consolidation: replay recent experiences
            recent_memories = self.memory.get_recent(5)
            if recent_memories and np.random.random() < 0.7:
                source = np.random.choice(recent_memories)
            else:
                # Random generation
                source = self._generate_random_scene()
        else:
            source = self._generate_random_scene()
            
        # Apply dream distortions
        distorted = self._apply_dream_distortions(source)
        
        # Add emotional tone
        emotional_valence = np.random.uniform(-1, 1)
        
        return {
            'cycle': cycle,
            'content': distorted,
            'emotional_valence': emotional_valence,
            'bizarreness': np.random.random(),
            'narrative_coherence': np.random.random()
        }
        
    def _apply_dream_distortions(self, content: Any) -> Any:
        """
        Apply characteristic dream distortions:
        - Bizarreness
        - Scene shifts
        - Identity transformations
        - Time distortion
        """
        distortions = [
            self._bizarre_transformation,
            self._scene_shift,
            self._identity_merge,
            self._temporal_distortion
        ]
        
        # Apply 1-3 random distortions
        n_distortions = np.random.randint(1, 4)
        selected = np.random.choice(distortions, n_distortions, replace=False)
        
        for distortion in selected:
            content = distortion(content)
            
        return content
6. Hypnosis Simulation
python
class HypnosisSimulator:
    """
    Simulate hypnotic states and suggestibility.
    """
    
    def __init__(self, state_space: AlteredStateSpace):
        self.state = state_space
        self.suggestibility = 0.5  # Baseline suggestibility
        self.trance_depth = 0.0
        self.suggestions = []
        
    def induction(self, technique: str = "progressive_relaxation",
                  duration_minutes: float = 10):
        """
        Simulate hypnotic induction.
        """
        # Gradually shift state toward hypnosis
        trajectory = []
        
        for minute in range(int(duration_minutes)):
            progress = minute / duration_minutes
            
            if technique == "progressive_relaxation":
                # Decrease arousal, increase sensory gating
                current = STATE_LIBRARY["waking"].copy()
                current['arousal'] = 1.0 - progress * 0.6
                current['sensory_gating'] = 0.5 + progress * 0.4
                
            elif technique == "eye_fixation":
                # Focused attention, then relaxation
                if progress < 0.5:
                    current = STATE_LIBRARY["focused_attention"].copy()
                else:
                    current = STATE_LIBRARY["hypnosis_light"].copy()
                    
            # Update state
            self.state.set_state(f"induction_{minute}", current)
            trajectory.append(self.state.dimensions.copy())
            
        # Set final hypnotic state
        self.state.set_state("hypnosis_light", STATE_LIBRARY["hypnosis_light"])
        self.trance_depth = 0.3
        
        return trajectory
        
    def deepen(self, technique: str = "fractionation", minutes: int = 5):
        """
        Deepen hypnotic trance.
        """
        for _ in range(minutes):
            if technique == "fractionation":
                # Alternate in and out of trance
                self.trance_depth += 0.1
                if self.trance_depth > 0.8:
                    self.state.set_state("hypnosis_deep", STATE_LIBRARY["hypnosis_deep"])
                    
        return self.trance_depth
        
    def give_suggestion(self, suggestion: str, type: str = "direct"):
        """
        Give hypnotic suggestion.
        """
        # Suggestion effectiveness depends on trance depth and suggestibility
        effectiveness = self.trance_depth * self.suggestibility
        
        # Different suggestion types
        if type == "direct":
            # Direct commands
            compliance_probability = effectiveness
            
        elif type == "indirect":
            # Permissive suggestions
            compliance_probability = effectiveness * 0.8
            
        elif type == "post_hypnotic":
            # Suggestions for after hypnosis
            compliance_probability = effectiveness * 0.6
            
        # Store suggestion
        self.suggestions.append({
            'content': suggestion,
            'type': type,
            'effectiveness': effectiveness,
            'time': len(self.suggestions)
        })
        
        return compliance_probability
7. Near-Death Experience Simulation
python
class NearDeathExperienceSimulator:
    """
    Simulate near-death experience phenomenology.
    """
    
    def __init__(self, state_space: AlteredStateSpace):
        self.state = state_space
        
    def experience(self, severity: float = 1.0):
        """
        Simulate NDE sequence.
        
        Common elements:
        - Out-of-body experience
        - Tunnel and light
        - Life review
        - Peace/euphoria
        - Border/decision point
        - Return to body
        """
        phases = [
            ('obesity', 0.1, self._out_of_body),
            ('tunnel', 0.2, self._tunnel_experience),
            ('life_review', 0.3, self._life_review),
            ('peace', 0.2, self._peace_state),
            ('border', 0.1, self._border_experience),
            ('return', 0.1, self._return_to_body)
        ]
        
        total_duration = sum(d for _, d, _ in phases)
        experience = []
        
        for phase_name, duration, phase_func in phases:
            # Scale duration by severity
            phase_duration = duration * total_duration * severity
            
            # Generate phase content
            phase_content = phase_func(phase_duration)
            experience.append({
                'phase': phase_name,
                'duration': phase_duration,
                'content': phase_content
            })
            
        return experience
        
    def _out_of_body(self, duration: float) -> Dict:
        """Simulate out-of-body experience."""
        self.state.set_state("obesity", {
            'self_boundary': 0.3,
            'awareness': 1.3,
            'coherence': 0.7
        })
        
        return {
            'vantage_point': 'above_body',
            'visual_accuracy': 0.8,
            'float_sensation': True
        }
        
    def _tunnel_experience(self, duration: float) -> Dict:
        """Simulate tunnel and light."""
        return {
            'tunnel_present': True,
            'light_brightness': 0.5 + duration,
            'light_warmth': 0.7,
            'movement_sensation': 'toward_light'
        }
        
    def _life_review(self, duration: float) -> Dict:
        """Simulate panoramic life review."""
        return {
            'temporal_scope': 'entire_life',
            'perspective': 'third_person',
            'emotional_impact': 0.9,
            'thematic_focus': 'relationships'
        }
8. Pathological State Analysis
python
class PathologicalStateAnalyzer:
    """
    Analyze and classify pathological states of consciousness.
    """
    
    def __init__(self, state_space: AlteredStateSpace):
        self.state = state_space
        self.diagnostic_history = []
        
    def assess(self, patient_data: Dict) -> Dict:
        """
        Assess consciousness state based on patient data.
        """
        # Extract state dimensions from data
        arousal = patient_data.get('eeg_arousal', 1.0)
        awareness = patient_data.get('behavioral_awareness', 1.0)
        coherence = patient_data.get('eeg_coherence', 1.0)
        
        # Set state
        self.state.set_state("patient_current", {
            'arousal': arousal,
            'awareness': awareness,
            'coherence': coherence
        })
        
        # Compare to pathological profiles
        distances = {}
        for pathology, profile in STATE_LIBRARY.items():
            if pathology in ['schizophrenia_acute', 'depression_severe', 'mania']:
                dist = 0
                for dim in ['arousal', 'awareness', 'coherence']:
                    dist += (self.state.dimensions[dim] - profile.get(dim, 0))**2
                distances[pathology] = np.sqrt(dist)
                
        # Find closest match
        if distances:
            closest = min(distances, key=distances.get)
            confidence = 1.0 / (1.0 + distances[closest])
            
            diagnosis = {
                'primary_diagnosis': closest,
                'confidence': confidence,
                'distances': distances,
                'state_dimensions': self.state.dimensions.copy()
            }
            
            self.diagnostic_history.append(diagnosis)
            return diagnosis
            
        return None
Integration with Consciousness Frameworks
python
class AlteredStatesIntegration:
    """
    Integrate altered states with other consciousness frameworks.
    """
    
    def __init__(self, state_space: AlteredStateSpace,
                 iit_system=None,
                 gwt_workspace=None,
                 qualia_space=None):
        self.state = state_space
        self.iit = iit_system
        self.gwt = gwt_workspace
        self.qualia = qualia_space
        
    def compute_iit_during_state(self, state_name: str) -> List[float]:
        """
        Compute Φ trajectory during altered state.
        """
        if self.iit is None:
            return None
            
        # Get state parameters
        state_params = STATE_LIBRARY.get(state_name, STATE_LIBRARY["waking"])
        
        # Transition to state
        trajectory = self.state.transition_to(state_name, duration=10.0)
        
        # Compute Φ at each point
        phi_trajectory = []
        for state_point in trajectory:
            # Set IIT system to corresponding state
            system_state = self._state_to_iit_state(state_point)
            self.iit.set_state(system_state)
            
            # Compute Φ
            phi, _, _ = system_phi(self.iit)
            phi_trajectory.append(phi)
            
        return phi_trajectory
