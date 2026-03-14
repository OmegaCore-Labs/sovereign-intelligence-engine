# Quantum Chemistry — Architecture Specification

## Overview

Quantum Chemistry module provides comprehensive simulation of molecular electronic structure using quantum algorithms. Enables accurate prediction of molecular properties, reaction mechanisms, and material characteristics through hybrid classical-quantum methods.

## Molecular Hamiltonians

### 1. Electronic Structure Hamiltonian

**Second Quantized Form:**
H = ∑_pq h_pq a†_p a_q + ½ ∑_pqrs h_pqrs a†_p a†_q a_r a_s

text

Where:
- h_pq: One-electron integrals (kinetic + nuclear attraction)
- h_pqrs: Two-electron integrals (Coulomb repulsion)
- a†_p, a_q: Fermionic creation/annihilation operators

**Implementation:**
```python
class MolecularHamiltonian:
    def __init__(self, molecule: str, basis: str = "sto-3g"):
        self.molecule = molecule
        self.basis = basis
        self.n_electrons = self._get_electron_count()
        self.n_orbitals = self._get_orbital_count()
        
        # Compute integrals (via PySCF/Psi4 interface)
        self.one_electron = self._compute_one_electron()
        self.two_electron = self._compute_two_electron()
        
        # Transform to qubit operators
        self.qubit_hamiltonian = self._jordan_wigner_transform()
        
    def _jordan_wigner_transform(self):
        # Map fermionic operators to Pauli strings
        # Each fermionic mode -> qubit
        # a†_p -> (X_p - iY_p)/2 * Z_{p-1}...Z_0
        pass
2. Basis Sets
Supported Basis Sets:

Minimal: STO-3G, STO-6G

Split-valence: 3-21G, 6-31G, 6-311G

Correlation-consistent: cc-pVDZ, cc-pVTZ, cc-pVQZ

Augmented: aug-cc-pVDZ, etc.

Effective core potentials (ECPs)

Custom basis definition

3. Integral Transformations
Atomic orbital to molecular orbital

Cholesky decomposition for two-electron integrals

Density fitting for efficiency

Tensor hypercontraction

Quantum Chemistry Algorithms
1. Variational Quantum Eigensolver (VQE)
Ansatz Options:

UCCSD (Unitary Coupled Cluster Singles Doubles)
python
def build_uccsd(n_qubits: int, n_electrons: int, excitations: List):
    """
    Build UCCSD ansatz circuit.
    
    Args:
        n_qubits: Number of qubits (spin orbitals)
        n_electrons: Number of electrons
        excitations: List of (i,a) singles and (i,j,a,b) doubles
    
    Returns:
        Parameterized quantum circuit
    """
    circuit = QuantumCircuit(n_qubits)
    
    # Initialize HF state
    for i in range(n_electrons):
        circuit.x(i)  # Occupied orbitals
    
    # Singles excitations (i -> a)
    for i, a in singles:
        # exp(θ (a†_a a_i - h.c.))
        circuit = add_single_excitation(circuit, i, a)
    
    # Doubles excitations (i,j -> a,b)
    for i, j, a, b in doubles:
        circuit = add_double_excitation(circuit, i, j, a, b)
    
    return circuit
Hardware-Efficient Ansatz
python
def build_hea(n_qubits: int, depth: int = 5, entangling: str = "cnot"):
    """
    Hardware-efficient ansatz for near-term devices.
    
    Args:
        n_qubits: Number of qubits
        depth: Number of layers
        entangling: "cnot", "cz", "iswap"
    """
    circuit = QuantumCircuit(n_qubits)
    
    for layer in range(depth):
        # Single-qubit rotations (variational)
        for q in range(n_qubits):
            circuit.rx(Parameter(f"θ_{layer}_{q}_x"), q)
            circuit.rz(Parameter(f"θ_{layer}_{q}_z"), q)
        
        # Entangling layer
        for q in range(0, n_qubits-1, 2):
            circuit.cx(q, q+1)
        for q in range(1, n_qubits-1, 2):
            circuit.cx(q, q+1)
    
    return circuit
ADAPT-VQE
Iterative ansatz growth

Select operators from pool based on gradient

Optimal for minimal circuit depth

2. Quantum Phase Estimation (QPE)
Purpose: Exact ground and excited state energies

Implementation:

python
def qpe_chemistry(hamiltonian, n_ancilla: int = 10):
    """
    Quantum phase estimation for molecular energy.
    
    Args:
        hamiltonian: Molecular Hamiltonian
        n_ancilla: Number of phase estimation qubits
    
    Returns:
        Energy estimate with Heisenberg-limited precision
    """
    # Initialize HF state
    hf_state = prepare_hartree_fock()
    
    # Build time evolution circuit
    U = trotter_evolution(hamiltonian, time=2*np.pi)
    
    # Controlled-U operations
    for i in range(n_ancilla):
        for j in range(2**i):
            circuit.append(U.control(), [i] + list(range(n_ancilla, n_ancilla+n_qubits)))
    
    # Inverse QFT on ancilla
    circuit.append(QFT(n_ancilla).inverse(), range(n_ancilla))
    
    # Measure and estimate phase
    measurements = execute(circuit, shots=1000)
    phase = estimate_phase(measurements)
    energy = 2*np.pi*phase / time
    
    return energy
3. Quantum Subspace Expansion
Purpose: Excited states without additional quantum resources

Implementation:

python
def subspace_expansion(ground_state_circuit, operators):
    """
    Compute excited states from ground state and operator pool.
    
    Args:
        ground_state_circuit: Circuit preparing ground state
        operators: List of excitation operators
    
    Returns:
        Excited state energies
    """
    # Measure ground state expectation values
    ground_state = execute(ground_state_circuit)
    
    # Build overlap and Hamiltonian matrices
    S = np.zeros((len(operators)+1, len(operators)+1))
    H = np.zeros_like(S)
    
    for i, op_i in enumerate(operators):
        for j, op_j in enumerate(operators):
            # ⟨ψ| op_i† op_j |ψ⟩
            S[i+1, j+1] = measure_overlap(ground_state, op_i, op_j)
            # ⟨ψ| op_i† H op_j |ψ⟩
            H[i+1, j+1] = measure_energy_expectation(ground_state, op_i, op_j)
    
    # Solve generalized eigenvalue problem
    energies = solve_generalized_eigenvalue(H, S)
    return energies[1:]  # Excited states
Error Mitigation for Chemistry
1. Zero-Noise Extrapolation (ZNE)
Scale noise by pulse stretching

Extrapolate to zero noise limit

2. Symmetry Verification
Particle number conservation

Total spin conservation

Post-select on correct symmetries

3. Measurement Error Mitigation
Calibration matrices

Iterative Bayesian unfolding

4. Hamiltonian Averaging
Twirling Pauli errors

Randomized compiling

Classical Pre-processing
1. Hartree-Fock
Restricted/Unrestricted HF

Closed/open shell

Convergence acceleration (DIIS)

2. Active Space Selection
CASSCF natural orbitals

Entanglement-based selection

Automated active space

3. Integral Compression
Tensor hypercontraction

Low-rank approximations

Double factorization

Property Calculation
1. Potential Energy Surfaces
Bond dissociation curves

Reaction coordinates

Conical intersections

2. Spectroscopic Properties
Vibrational frequencies

IR intensities

Raman activities

NMR chemical shifts

3. Electronic Properties
Dipole moments

Polarizabilities

Hyperpolarizabilities

Ionization potentials

Electron affinities

4. Non-adiabatic Dynamics
Surface hopping

Fewest switches

Ehrenfest dynamics

Integration with ARIS
python
# Example: Complete quantum chemistry workflow
from aris.quantum.chemistry import Molecule, VQE, QPE
from aris.quantum.hardware import HardwareModel

# Define molecule
h2o = Molecule.from_xyz("h2o.xyz")
h2o.basis = "cc-pVDZ"
h2o.charge = 0
h2o.spin = 0  # Singlet

# Run VQE for ground state
vqe = VQE(ansatz="UCCSD", optimizer="L-BFGS-B")
ground_state = vqe.optimize(h2o)
print(f"Ground state energy: {ground_state.energy} Ha")

# Run QPE for higher precision
qpe = QPE(n_ancilla=12, precision=1e-6)
exact_energy = qpe.run(h2o, initial_state=ground_state.circuit)
print(f"Exact energy: {exact_energy} Ha")

# Compute excited states
excitations = ground_state.excited_states(n_states=5)
for i, state in enumerate(excitations):
    print(f"State {i+1}: {state.energy} Ha, {state.oscillator_strength}")

# Simulate spectroscopy
ir_spectrum = h2o.vibrational_spectrum(ground_state)
uv_vis_spectrum = h2o.electronic_spectrum(excitations)

# Analyze with classical tools
from aris.chemistry import Analysis
density = Analysis.electron_density(ground_state)
bond_order = Analysis.wiberg_bond_indices(ground_state)
Benchmark Molecules
Molecule	Basis	Method	Energy (Ha)	Accuracy	Qubits
H₂	STO-3G	FCI	-1.137	Exact	2
H₂	cc-pVQZ	FCI	-1.174	Exact	28
LiH	STO-3G	UCCSD	-7.882	0.001	12
BeH₂	cc-pVDZ	UCCSD	-15.775	0.002	28
H₂O	STO-3G	UCCSD	-75.012	0.005	14
H₂O	cc-pVDZ	UCCSD	-76.124	0.003	28
NH₃	cc-pVDZ	UCCSD	-56.451	0.004	30
C₂H₄	STO-3G	UCCSD	-77.074	0.010	28
