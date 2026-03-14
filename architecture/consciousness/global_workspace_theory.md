# Global Workspace Theory (GWT) — Architecture Specification

## Overview

Global Workspace Theory (GWT) proposes that consciousness functions as a global workspace where information from specialized processors is broadcast globally to the rest of the system. This module implements computational models of GWT, enabling simulation of conscious access, attention, and global broadcast dynamics.

## Core Architecture

### 1. Workspace Components

```python
class GlobalWorkspace:
    """
    Central workspace for conscious access and broadcast.
    """
    
    def __init__(self, workspace_capacity: int = 4, n_processors: int = 100):
        self.capacity = workspace_capacity  # Number of items in workspace
        self.workspace = []  # Current workspace contents
        self.processors = [SpecializedProcessor(i) for i in range(n_processors)]
        self.broadcast_history = []
        self.attention_weights = np.ones(n_processors) / n_processors
        
    def step(self, inputs: Dict[int, np.ndarray]) -> Dict:
        """
        Single timestep of workspace dynamics.
        
        1. Processors compute local results
        2. Competition for workspace access
        3. Global broadcast of winning contents
        4. Processors receive broadcast
        """
        # Step 1: Parallel processor computation
        processor_outputs = {}
        for pid, processor in enumerate(self.processors):
            if pid in inputs:
                processor_outputs[pid] = processor.process(inputs[pid])
            else:
                processor_outputs[pid] = processor.idle_output()
        
        # Step 2: Competition for workspace
        candidates = self._compute_candidates(processor_outputs)
        workspace_updates = self._competition(candidates)
        
        # Step 3: Update workspace
        self.workspace = self._update_workspace(workspace_updates)
        
        # Step 4: Global broadcast
        broadcast = self._format_broadcast(self.workspace)
        for processor in self.processors:
            processor.receive_broadcast(broadcast)
            
        self.broadcast_history.append(broadcast)
        
        return {
            'workspace': self.workspace.copy(),
            'broadcast': broadcast,
            'attention': self.attention_weights.copy()
        }
2. Specialized Processors
python
class SpecializedProcessor:
    """
    Domain-specific processor module.
    
    Examples: Visual cortex area V4, auditory cortex, prefrontal cortex,
    language areas, etc.
    """
    
    def __init__(self, processor_id: int, domain: str = "general"):
        self.id = processor_id
        self.domain = domain
        self.specialized_knowledge = self._initialize_knowledge()
        self.recurrent_connections = []  # Connections to other processors
        self.current_activation = 0.0
        self.broadcast_queue = []  # Recent broadcasts received
        
    def process(self, input_data: np.ndarray) -> ProcessorOutput:
        """
        Process input data using specialized algorithms.
        """
        # Domain-specific processing
        if self.domain == "visual":
            features = self._extract_visual_features(input_data)
        elif self.domain == "auditory":
            features = self._extract_auditory_features(input_data)
        else:
            features = self._general_processing(input_data)
            
        # Compute activation (competition strength)
        self.current_activation = self._compute_activation(features)
        
        return ProcessorOutput(
            processor_id=self.id,
            features=features,
            activation=self.current_activation,
            confidence=self._compute_confidence(features)
        )
        
    def receive_broadcast(self, broadcast: Broadcast):
        """
        Receive global broadcast from workspace.
        """
        self.broadcast_queue.append(broadcast)
        
        # Process broadcast (may trigger learning, priming, etc.)
        if len(self.broadcast_queue) > 10:
            self.broadcast_queue.pop(0)
            
        # Update based on broadcast (e.g., priming)
        self._update_from_broadcast(broadcast)
3. Competition Dynamics
python
class WorkspaceCompetition:
    """
    Competition for access to global workspace.
    """
    
    def __init__(self, workspace_capacity: int, inhibition_strength: float = 0.5):
        self.capacity = workspace_capacity
        self.inhibition = inhibition_strength
        
    def compete(self, candidates: List[ProcessorOutput]) -> List[ProcessorOutput]:
        """
        Winner-take-all competition with mutual inhibition.
        
        Implements biased competition model where:
        - Processors inhibit each other
        - Top-down attention biases competition
        - Recurrent excitation supports coalitions
        """
        n = len(candidates)
        activations = np.array([c.activation for c in candidates])
        confidence = np.array([c.confidence for c in candidates])
        
        # Mutual inhibition
        inhibition_matrix = np.ones((n, n)) * self.inhibition
        np.fill_diagonal(inhibition_matrix, 0)
        
        net_activation = activations - np.sum(inhibition_matrix * activations, axis=1)
        
        # Bias by confidence (self-reinforcement)
        net_activation *= (1 + confidence)
        
        # Top-down attention bias
        net_activation *= self.attention_weights
        
        # Select winners
        winners_idx = np.argsort(net_activation)[-self.capacity:]
        
        return [candidates[i] for i in winners_idx]
4. Attention Mechanisms
python
class AttentionSystem:
    """
    Top-down attention modulation of competition.
    """
    
    def __init__(self):
        self.attention_goals = []  # Current attention goals
        self.salience_map = None  # Bottom-up salience
        
    def compute_attention_weights(self, task_demand: str, 
                                  processors: List[SpecializedProcessor]) -> np.ndarray:
        """
        Compute attention weights based on task demands.
        """
        weights = np.zeros(len(processors))
        
        if task_demand == "visual_search":
            # Weight visual processors higher
            for i, p in enumerate(processors):
                if p.domain in ["visual", "frontal_eye_field"]:
                    weights[i] = 1.0
                    
        elif task_demand == "auditory_streaming":
            for i, p in enumerate(processors):
                if p.domain in ["auditory", "temporal"]:
                    weights[i] = 1.0
                    
        elif task_demand == "working_memory":
            for i, p in enumerate(processors):
                if p.domain in ["prefrontal", "parietal"]:
                    weights[i] = 1.0
                    
        # Normalize
        weights = weights / (weights.sum() + 1e-8)
        
        return weights
5. Global Broadcast
python
class GlobalBroadcast:
    """
    Global availability of workspace contents.
    """
    
    def __init__(self, broadcast_format: str = "compressed"):
        self.format = broadcast_format
        
    def broadcast(self, workspace_contents: List, processors: List[SpecializedProcessor]):
        """
        Broadcast workspace contents to all processors.
        """
        # Format broadcast
        if self.format == "compressed":
            # Send only essential features
            broadcast = self._compress_broadcast(workspace_contents)
        elif self.format == "rich":
            # Send full workspace representation
            broadcast = self._rich_broadcast(workspace_contents)
            
        # Distribute to all processors
        for processor in processors:
            processor.receive_broadcast(broadcast)
            
        return broadcast
6. Conscious Access Simulation
python
class ConsciousAccessSimulator:
    """
    Simulate conscious access to specific information.
    """
    
    def __init__(self, workspace: GlobalWorkspace):
        self.workspace = workspace
        self.access_history = []
        
    def access_information(self, information_id: int, duration: float = 0.5):
        """
        Simulate conscious access to specific information.
        
        Information becomes:
        1. Available for verbal report
        2. Available for voluntary action
        3. Encoding in episodic memory
        4. Available to multiple processors
        """
        # Step 1: Boost activation of relevant processors
        for processor in self.workspace.processors:
            if processor.id == information_id:
                processor.current_activation *= 2.0
                
        # Step 2: Run workspace dynamics
        results = []
        timesteps = int(duration * 100)  # 10ms timesteps
        for t in range(timesteps):
            step_result = self.workspace.step({})
            results.append(step_result)
            
            # Check if information entered workspace
            if any(item.id == information_id for item in step_result['workspace']):
                self.access_history.append({
                    'time': t,
                    'information': information_id,
                    'workspace_context': step_result['workspace']
                })
                
        return results
Empirical Validation
1. Binocular Rivalry Simulation
python
def simulate_binocular_rivalry(workspace: GlobalWorkspace, duration: float = 10.0):
    """
    Simulate binocular rivalry dynamics.
    
    Two incompatible percepts compete for conscious access.
    """
    # Left eye processor, Right eye processor
    left = workspace.processors[0]  # Assume configured as left visual
    right = workspace.processors[1]  # Assume configured as right visual
    
    # Continuous input to both
    input_signal = np.ones(100)  # Constant input
    
    results = []
    for t in range(int(duration * 100)):
        # Alternate dominance and suppression
        workspace.step({
            0: input_signal,  # Left eye
            1: input_signal   # Right eye
        })
        
        # Track which percept is in workspace
        if any(item.processor_id == 0 for item in workspace.workspace):
            results.append('left')
        elif any(item.processor_id == 1 for item in workspace.workspace):
            results.append('right')
        else:
            results.append('mixed')
            
    # Should show alternating dominance periods
    return results
2. Attentional Blink
python
def simulate_attentional_blink(workspace: GlobalWorkspace):
    """
    Simulate attentional blink phenomenon.
    
    Second target is missed if presented 200-500ms after first.
    """
    # Rapid serial visual presentation (RSVP) simulation
    stream = ['T1', 'D1', 'D2', 'T2', 'D3', 'D4']
    
    accuracy = []
    for lag in [100, 200, 300, 400, 500, 600]:  # ms
        # Present stream with T2 at given lag
        results = []
        for item in stream:
            # Input to visual processors
            workspace.step({0: item})
            
            if item == 'T2':
                # Check if T2 reached workspace
                detected = any(getattr(w, 'id', None) == 'T2' for w in workspace.workspace)
                results.append(detected)
                
        accuracy.append(np.mean(results))
        
    # Should show dip at 200-500ms
    return accuracy
Integration with Neural Networks
python
class NeuralGWT(GlobalWorkspace):
    """
    GWT implemented with neural network components.
    """
    
    def __init__(self, n_units: int = 1000, workspace_units: int = 100):
        super().__init__()
        
        # Neural implementation
        self.processor_networks = [
            self._build_processor_network() for _ in range(n_processors)
        ]
        self.workspace_network = self._build_workspace_network(workspace_units)
        self.broadcast_weights = self._initialize_broadcast_weights()
        
    def _build_processor_network(self) -> torch.nn.Module:
        """Build specialized processor as neural network."""
        return torch.nn.Sequential(
            torch.nn.Linear(100, 200),
            torch.nn.ReLU(),
            torch.nn.Linear(200, 50),
            torch.nn.Softmax(dim=-1)
        )
        
    def _build_workspace_network(self, units: int) -> torch.nn.Module:
        """Build workspace as recurrent neural network."""
        return torch.nn.GRU(50, units, batch_first=True)
        
    def neural_step(self, inputs: torch.Tensor) -> Dict:
        """
        Neural implementation of workspace dynamics.
        """
        # Process inputs through processor networks
        processor_activations = []
        for i, network in enumerate(self.processor_networks):
            if i < inputs.shape[0]:
                act = network(inputs[i])
                processor_activations.append(act)
                
        # Competition via lateral inhibition
        processor_tensor = torch.stack(processor_activations)
        inhibited = self._lateral_inhibition(processor_tensor)
        
        # Workspace update
        workspace_out, _ = self.workspace_network(inhibited.unsqueeze(0))
        
        # Broadcast via learned weights
        broadcast = torch.mm(workspace_out.squeeze(0), self.broadcast_weights)
        
        return {
            'processor_activations': processor_activations,
            'workspace_state': workspace_out,
            'broadcast': broadcast
        }
