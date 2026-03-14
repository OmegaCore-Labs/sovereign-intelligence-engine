# Predictive Processing — Architecture Specification

## Overview

Predictive Processing (PP) framework, also known as the Free Energy Principle, proposes that the brain implements hierarchical generative models that predict sensory input, with perception emerging from the minimization of prediction error. This module implements computational models of predictive processing, active inference, and precision weighting.

## Core Architecture

### 1. Hierarchical Generative Model

```python
class HierarchicalGenerativeModel:
    """
    Multi-level generative model of sensory causes.
    
    Each level predicts the activity at the level below,
    with top-down predictions and bottom-up prediction errors.
    """
    
    def __init__(self, layer_dims: List[int]):
        """
        Initialize hierarchical model.
        
        Args:
            layer_dims: List of dimensions for each layer
                       [sensory_input_dim, layer1_dim, layer2_dim, ...]
        """
        self.n_layers = len(layer_dims)
        self.dims = layer_dims
        
        # Generative parameters (top-down)
        self.generative_weights = []
        self.generative_biases = []
        
        # Recognition parameters (bottom-up)
        self.recognition_weights = []
        self.recognition_biases = []
        
        # Current activations
        self.predictions = [np.zeros(dim) for dim in layer_dims]
        self.prediction_errors = [np.zeros(dim) for dim in layer_dims]
        self.posteriors = [np.zeros(dim) for dim in layer_dims]
        
        # Precision parameters (inverse variance)
        self.precisions = [np.ones(dim) for dim in layer_dims]
        
        self._initialize_parameters()
        
    def _initialize_parameters(self):
        """Initialize weight matrices and biases."""
        for i in range(self.n_layers - 1):
            # Top-down connections (higher -> lower)
            w_gen = np.random.randn(self.dims[i+1], self.dims[i]) * 0.1
            self.generative_weights.append(w_gen)
            self.generative_biases.append(np.zeros(self.dims[i]))
            
            # Bottom-up connections (lower -> higher)
            w_rec = np.random.randn(self.dims[i], self.dims[i+1]) * 0.1
            self.recognition_weights.append(w_rec)
            self.recognition_biases.append(np.zeros(self.dims[i+1]))
2. Prediction Error Computation
python
class PredictionError:
    """
    Compute prediction errors at each hierarchical level.
    """
    
    @staticmethod
    def compute(sensory_input: np.ndarray, 
                prediction: np.ndarray,
                precision: np.ndarray) -> np.ndarray:
        """
        Compute precision-weighted prediction error.
        
        ε = precision * (input - prediction)
        """
        error = sensory_input - prediction
        weighted_error = precision * error
        return weighted_error
        
    @staticmethod
    def precision_weighted_energy(error: np.ndarray, 
                                   precision: np.ndarray) -> float:
        """
        Compute free energy contribution from prediction error.
        
        F = 0.5 * ε^T * precision * ε - 0.5 * log(det(precision))
        """
        weighted_error = precision * error
        energy = 0.5 * np.dot(error, weighted_error)
        entropy_penalty = -0.5 * np.sum(np.log(precision + 1e-8))
        return energy + entropy_penalty
3. Hierarchical Inference
python
class PredictiveInference:
    """
    Perform hierarchical inference via prediction error minimization.
    """
    
    def __init__(self, model: HierarchicalGenerativeModel):
        self.model = model
        self.learning_rate = 0.01
        
    def infer(self, sensory_input: np.ndarray, 
              n_iterations: int = 100) -> Dict:
        """
        Perform perceptual inference.
        
        Iteratively update posterior expectations to minimize
        prediction error at all levels.
        """
        # Initialize with prior expectations
        for level in range(self.model.n_layers):
            if level == 0:
                # Bottom layer gets sensory input
                self.model.posteriors[level] = sensory_input.copy()
            else:
                # Higher layers start with zeros (or prior)
                self.model.posteriors[level] = np.zeros(self.model.dims[level])
                
        # Inference loop
        free_energy_history = []
        
        for iteration in range(n_iterations):
            # Forward pass (bottom-up)
            self._forward_predictions()
            
            # Compute prediction errors
            total_fe = 0
            for level in range(self.model.n_layers):
                if level == 0:
                    # Sensory level error
                    error = PredictionError.compute(
                        sensory_input,
                        self.model.predictions[0],
                        self.model.precisions[0]
                    )
                else:
                    # Higher level error
                    error = PredictionError.compute(
                        self.model.posteriors[level],
                        self.model.predictions[level],
                        self.model.precisions[level]
                    )
                    
                self.model.prediction_errors[level] = error
                total_fe += PredictionError.precision_weighted_energy(
                    error, self.model.precisions[level]
                )
                
            # Backward pass (top-down) to update posteriors
            self._backward_update()
            
            free_energy_history.append(total_fe)
            
        return {
            'posteriors': self.model.posteriors.copy(),
            'predictions': self.model.predictions.copy(),
            'free_energy': free_energy_history,
            'final_fe': free_energy_history[-1]
        }
        
    def _forward_predictions(self):
        """Generate top-down predictions at each level."""
        # Start from top
        for level in range(self.model.n_layers - 1, 0, -1):
            # Generate prediction for level below
            w = self.model.generative_weights[level-1]
            b = self.model.generative_biases[level-1]
            
            prediction = np.dot(w, self.model.posteriors[level]) + b
            self.model.predictions[level-1] = prediction
            
    def _backward_update(self):
        """Update posteriors based on prediction errors."""
        # Start from bottom
        for level in range(self.model.n_layers - 1):
            # Combine bottom-up error and top-down prediction
            bottom_up = np.dot(
                self.model.recognition_weights[level].T,
                self.model.prediction_errors[level]
            )
            
            if level < self.model.n_layers - 1:
                top_down = self.model.prediction_errors[level + 1]
            else:
                top_down = 0
                
            # Update posterior
            gradient = bottom_up + top_down
            self.model.posteriors[level + 1] -= self.learning_rate * gradient
4. Precision Optimization
python
class PrecisionOptimizer:
    """
    Optimize precision (inverse variance) parameters.
    
    Precision determines the weight given to prediction errors
    at each level and can be modulated by attention.
    """
    
    def __init__(self, model: HierarchicalGenerativeModel):
        self.model = model
        self.precision_learning_rate = 0.001
        
    def update_precisions(self, prediction_errors: List[np.ndarray]):
        """
        Update precision estimates based on recent prediction errors.
        """
        for level, error in enumerate(prediction_errors):
            # Estimate variance from squared errors
            estimated_var = np.mean(error**2) + 1e-8
            
            # Update precision (inverse variance)
            target_precision = 1.0 / estimated_var
            
            # Gradient descent on precision
            precision_grad = 0.5 * (1.0/self.model.precisions[level] - estimated_var)
            self.model.precisions[level] -= self.precision_learning_rate * precision_grad
            
            # Ensure positive
            self.model.precisions[level] = np.maximum(
                self.model.precisions[level], 1e-8
            )
            
    def attention_modulation(self, attention_weights: np.ndarray):
        """
        Modulate precision based on attention.
        
        Attention increases precision (decreases variance)
        for attended features.
        """
        for level in range(self.model.n_layers):
            if level < len(attention_weights):
                # Attention increases precision multiplicatively
                self.model.precisions[level] *= (1.0 + attention_weights[level])
5. Active Inference
python
class ActiveInference:
    """
    Extend predictive processing to action selection.
    
    Agents act to minimize expected free energy by
    sampling sensory inputs that match predictions.
    """
    
    def __init__(self, model: HierarchicalGenerativeModel,
                 possible_actions: List[Callable]):
        self.model = model
        self.actions = possible_actions
        self.inference = PredictiveInference(model)
        
    def select_action(self, current_sensory: np.ndarray,
                      n_samples: int = 10) -> int:
        """
        Select action that minimizes expected free energy.
        """
        # Current state inference
        current_state = self.inference.infer(current_sensory)
        
        # Evaluate each possible action
        expected_fes = []
        
        for action_idx, action in enumerate(self.actions):
            # Simulate outcome of action
            expected_sensory = self._simulate_action_outcome(
                action, current_state['posteriors']
            )
            
            # Compute expected free energy
            efe = self._expected_free_energy(
                expected_sensory, current_state['posteriors']
            )
            expected_fes.append(efe)
            
        # Select action with minimum expected free energy
        best_action = np.argmin(expected_fes)
        
        return best_action
        
    def _simulate_action_outcome(self, action: Callable,
                                 current_beliefs: List[np.ndarray]) -> np.ndarray:
        """
        Predict sensory consequences of action.
        """
        # Apply action model to beliefs
        new_beliefs = action(current_beliefs)
        
        # Generate sensory prediction from new beliefs
        sensory_prediction = self._generate_sensory_prediction(new_beliefs)
        
        return sensory_prediction
        
    def _expected_free_energy(self, expected_sensory: np.ndarray,
                              beliefs: List[np.ndarray]) -> float:
        """
        Compute expected free energy of sensory outcome.
        
        EFE = -E[ln P(o|s)] - D_KL[Q(s) || P(s)]
        """
        # Ambiguity term: -E[ln P(o|s)]
        ambiguity = -np.mean(np.log(self._likelihood(expected_sensory, beliefs) + 1e-8))
        
        # Risk term: KL divergence from prior
        risk = self._kl_divergence(beliefs[-1], self._prior())
        
        return ambiguity + risk
        
    def _likelihood(self, sensory: np.ndarray, beliefs: List[np.ndarray]) -> float:
        """Compute likelihood of sensory given beliefs."""
        # Simplified: Gaussian likelihood
        prediction = self._generate_sensory_prediction(beliefs)
        error = sensory - prediction
        variance = np.var(error) + 1e-8
        return np.exp(-0.5 * np.sum(error**2) / variance)
6. Learning and Plasticity
python
class PredictiveLearning:
    """
    Learn generative model parameters through prediction error minimization.
    """
    
    def __init__(self, model: HierarchicalGenerativeModel):
        self.model = model
        self.learning_rate = 0.001
        self.memory = []  # Store recent experiences
        
    def update_parameters(self, sensory_sequence: List[np.ndarray]):
        """
        Update generative weights based on prediction errors.
        """
        gradients = self._compute_gradients(sensory_sequence)
        
        # Update weights
        for level in range(self.model.n_layers - 1):
            self.model.generative_weights[level] -= self.learning_rate * gradients['gen_w'][level]
            self.model.generative_biases[level] -= self.learning_rate * gradients['gen_b'][level]
            self.model.recognition_weights[level] -= self.learning_rate * gradients['rec_w'][level]
            self.model.recognition_biases[level] -= self.learning_rate * gradients['rec_b'][level]
            
    def _compute_gradients(self, sensory_sequence):
        """Compute parameter gradients via backpropagation through time."""
        # Simplified: accumulate prediction error gradients
        gradients = {
            'gen_w': [np.zeros_like(w) for w in self.model.generative_weights],
            'gen_b': [np.zeros_like(b) for b in self.model.generative_biases],
            'rec_w': [np.zeros_like(w) for w in self.model.recognition_weights],
            'rec_b': [np.zeros_like(b) for b in self.model.recognition_biases]
        }
        
        for t, sensory in enumerate(sensory_sequence):
            # Run inference
            inference = PredictiveInference(self.model)
            result = inference.infer(sensory, n_iterations=10)
            
            # Accumulate gradients (simplified)
            for level in range(self.model.n_layers - 1):
                # Hebbian-like update
                post_error = self.model.prediction_errors[level]
                post_upper = self.model.posteriors[level + 1]
                
                gradients['gen_w'][level] += np.outer(post_error, post_upper)
                gradients['gen_b'][level] += post_error
                
        return gradients
7. Illusions and Phenomena
python
class PredictivePhenomena:
    """
    Simulate perceptual phenomena explained by predictive processing.
    """
    
    def __init__(self, model: HierarchicalGenerativeModel):
        self.model = model
        self.inference = PredictiveInference(model)
        
    def binocular_rivalry(self, left_input: np.ndarray, 
                          right_input: np.ndarray,
                          duration: int = 100) -> List[np.ndarray]:
        """
        Simulate binocular rivalry.
        
        Incompatible predictions from two eyes compete.
        """
        perceptions = []
        
        for t in range(duration):
            # Alternate dominance
            if t % 20 < 10:  # First eye dominant
                sensory = left_input
                # Boost precision for left eye features
                self.model.precisions[0] *= 1.2
            else:  # Second eye dominant
                sensory = right_input
                # Boost precision for right eye features
                self.model.precisions[0] *= 1.2
                
            # Run inference
            result = self.inference.infer(sensory)
            perceptions.append(result['posteriors'][-1])  # Top-level percept
            
        return perceptions
        
    def mismatch_negativity(self, standard: np.ndarray, 
                            deviant: np.ndarray) -> np.ndarray:
        """
        Simulate mismatch negativity (MMN) ERP component.
        
        MMN occurs when sensory input violates predictions.
        """
        # Build prediction from sequence of standards
        standards = [standard] * 10
        for stim in standards:
            self.inference.infer(stim)
            
        # Present deviant and record prediction error
        result = self.inference.infer(deviant)
        mmn = self.model.prediction_errors[0]  # Sensory level error
        
        return mmn
Integration with Neural Networks
python
class DeepPredictiveNetwork:
    """
    Deep neural network implementation of predictive processing.
    """
    
    def __init__(self, layer_dims: List[int]):
        self.model = HierarchicalGenerativeModel(layer_dims)
        
        # Convert to PyTorch
        import torch
        import torch.nn as nn
        
        self.gen_networks = nn.ModuleList()
        self.rec_networks = nn.ModuleList()
        
        for i in range(len(layer_dims) - 1):
            self.gen_networks.append(
                nn.Linear(layer_dims[i+1], layer_dims[i])
            )
            self.rec_networks.append(
                nn.Linear(layer_dims[i], layer_dims[i+1])
            )
            
    def forward(self, sensory_input: torch.Tensor, 
                n_iterations: int = 10) -> Dict:
        """
        PyTorch forward pass with iterative inference.
        """
        batch_size = sensory_input.shape[0]
        
        # Initialize states
        states = [sensory_input]
        for dim in self.model.dims[1:]:
            states.append(torch.zeros(batch_size, dim))
            
        predictions = []
        errors = []
        
        for _ in range(n_iterations):
            # Top-down predictions
            preds = []
            for i in range(len(states) - 1, 0, -1):
                pred = self.gen_networks[i-1](states[i])
                preds.insert(0, pred)
                
            # Bottom-up errors
            errs = []
            for i, (state, pred) in enumerate(zip(states[:-1], preds)):
                err = state - pred
                errs.append(err)
                
                if i < len(states) - 1:
                    # Update higher state
                    states[i+1] = states[i+1] + self.rec_networks[i](err)
                    
            predictions.append(preds)
            errors.append(errs)
            
        return {
            'states': states,
            'predictions': predictions[-1],
            'errors': errors[-1]
        }
