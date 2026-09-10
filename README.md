# Spin-Resolved VQE-QSE Simulation of Ethylene

This repository contains the research notebook and final report for a quantum-chemistry study developed for the **Alexandria Quantum Hackathon 2026**. The project combines the Variational Quantum Eigensolver (VQE) with Quantum Subspace Expansion (QSE) to investigate the ground and excited states of planar and 90-degree-twisted ethylene.

The workflow validates the mapped qubit Hamiltonians against independent PySCF CASCI calculations, compares chemically motivated and hardware-efficient ansatze, studies QSE state selection and conditioning, benchmarks noise-mitigation methods, and evaluates selected symmetry-tapered circuits on IBM Quantum hardware.

> **Research status:** This is a validated proof of principle within the selected active spaces and basis set. It is not a quantitatively converged prediction of the experimental ethylene spectrum. The PDF report contains the final audited interpretation of the results.

## Repository contents

| File | Description |
| --- | --- |
| [`ethylene_vqse_AQH.ipynb`](ethylene_vqse_AQH.ipynb) | Executed Jupyter notebook containing molecular construction, Hamiltonian mapping, VQE, QSE, noise simulation, IBM Quantum execution, classical benchmarks, figures, and a butadiene extension. |
| [`Report.pdf`](Report.pdf) | Three-page final report with the audited methodology, results, limitations, and references. |

## Scientific workflow

1. Construct planar and rigidly twisted ethylene geometries.
2. Generate STO-3G electronic-structure problems with PySCF and Qiskit Nature.
3. Build CAS(2,2) and CAS(2,4) active-space Hamiltonians.
4. Validate determinant-space CASCI energies against exact qubit-Hamiltonian diagonalization.
5. Compare UCCSD and a shallow hardware-efficient ansatz using classical optimization.
6. Recover excited states with QSE using single and double excitation operators.
7. Label roots using energy, particle number, spin, transition strength, and state continuity rather than array index alone.
8. Evaluate readout mitigation and linear zero-noise extrapolation under shot noise and a representative device-noise model.
9. Execute fixed, symmetry-tapered energy-estimation circuits on IBM Fez.
10. Test extensibility with a CAS(4,4) butadiene example.

## Main findings

- Independent CASCI and qubit-space diagonalization agree to approximately $10^{-13}\,E_h$, validating the active-space construction, energy shifts, orbital ordering, and mapping.
- Ideal UCCSD reproduces all four target singlet energies with a maximum error of approximately $1.24\times10^{-8}\,E_h$.
- The shallow hardware-efficient ansatz is accurate for CAS(2,2) but is not sufficiently expressive for CAS(2,4).
- State-matched ideal QSE recovers the selected active-space excitations to the displayed numerical precision.
- The planar bright $\pi\rightarrow\pi^*$ gap is approximately 14.16 eV in STO-3G, far above the experimental band near 7.6 eV. The basis-set audit attributes most of this discrepancy to missing diffuse radial flexibility, not to QSE itself.
- At 90 degrees, the active-space global ground state is a triplet, approximately 0.0696 eV below the lowest singlet. Optical analysis therefore requires a separate singlet reference.
- The butadiene pilot demonstrates the importance of double excitations: the singles-plus-doubles pool recovers a low dark singlet at 7.659 eV that the singles-only low-root analysis misses.

### Audited active-space spectrum

| Geometry | Active space | Tapered qubits / Pauli terms | Lowest energy ($E_h$) | First reported gap (eV) | Selected singlet gap (eV) |
| --- | ---: | ---: | ---: | ---: | ---: |
| Planar | CAS(2,2) | 1 / 15 | -77.116592 | 4.7919 | 14.1546 |
| Planar | CAS(2,4) | 3 / 61 | -77.116777 | 4.7969 | 14.1596 |
| Twisted | CAS(2,2) | 2 / 27 | -76.979487 | 0.0696 | 6.5802 |
| Twisted | CAS(2,4) | 5 / 185 | -76.979487 | 0.0696 | 6.5235 |

For the planar models, the first gap is the lowest triplet gap and the selected singlet gap is the bright transition from $S_0$. For the twisted models, the first gap is $S_0-T_0$ and the selected singlet gap is $S_1-S_0$.

### Audited IBM Fez results

The hardware experiment used 72 PUBs, 294,912 total shots, and three repeats of 4,096 shots per circuit. Parameters were optimized classically before submission, so this is fixed-circuit energy estimation rather than hardware-in-the-loop VQE or hardware QSE.

Signed energy errors relative to exact diagonalization of the same active-space target are:

| Target | Raw ($\mathrm{m}E_h$) | Readout-mitigated ($\mathrm{m}E_h$) | Linear ZNE ($\mathrm{m}E_h$) |
| --- | ---: | ---: | ---: |
| Planar singlet | +7.53 +/- 1.83 | +1.74 +/- 1.47 | +2.01 +/- 1.06 |
| Twisted triplet | +12.44 +/- 0.18 | +6.06 +/- 0.73 | -2.55 +/- 1.48 |

The reported spreads are population standard deviations over three repeats from one hardware job; they are not confidence intervals or a calibration-drift study. The twisted linear-ZNE estimate falls below the exact target and is therefore nonvariational.

## Environment

The recorded execution environment was:

| Package | Version |
| --- | ---: |
| Python | 3.x |
| Qiskit | 2.5.2 |
| Qiskit Nature | 0.8.0 |
| Qiskit Algorithms | 0.4.0 |
| Qiskit Aer | 0.17.2 |
| Qiskit IBM Runtime | 0.49.0 |
| PySCF | 2.14.0 |
| NumPy | 2.1.3 |
| SciPy | 1.16.3 |

To create a local environment:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install \
  "qiskit==2.5.2" \
  "qiskit-nature==0.8.0" \
  "qiskit-algorithms==0.4.0" \
  "qiskit-aer==0.17.2" \
  "qiskit-ibm-runtime==0.49.0" \
  "pyscf==2.14.0" \
  "numpy==2.1.3" \
  "scipy==1.16.3" \
  matplotlib jupyterlab
```

On Windows PowerShell, activate the environment with:

```powershell
.\.venv\Scripts\Activate.ps1
```

## Running the notebook

Start Jupyter and open the executed notebook:

```bash
jupyter lab "ethylene_vqse_AQH.ipynb"
```

Run the cells in order. The full workflow includes several classical electronic-structure calculations and the butadiene pilot, so execution time depends strongly on the machine and can exceed 30 minutes.

### IBM Quantum execution

The notebook reads IBM credentials from the `IBM_QUANTUM_TOKEN` environment variable or a Google Colab secret with the same name. The token is never intentionally printed or stored in the notebook.

> **Important:** The uploaded notebook currently sets `RUN_ON_HARDWARE = True`. Change it to `False` before using **Run all** unless you deliberately intend to submit a new hardware job. A new execution dynamically selects the least-busy eligible backend and therefore may not run on IBM Fez or reproduce the archived hardware values.

Never commit an API token, `.env` file, or local Qiskit account configuration.

## Reproducibility and audit note

The calculations use seed `42` unless a shot-seed sweep is stated, and the conversion factor is $1\,E_h=27.211386245988$ eV.

The PDF is the authoritative, post-run-audited record. Two outputs in the currently executed notebook predate the final audit:

1. The twisted CAS(2,4) selector displays a neighboring 6.557 eV root; state matching gives the final $S_1-S_0$ value of 6.523527 eV.
2. The IBM hardware table printed in the notebook differs from the final audited Table 2 in the report. The report excludes an observable-layout error in an earlier noise branch and a root-index mismatch in a noisy-QSE branch.

For a fully self-contained reproducibility release, the notebook should be updated to implement the final state-matching and hardware-audit path, then rerun from a clean environment. Until that update is committed, cite the numerical claims from `Report.pdf` and treat affected notebook cells as pre-audit provenance.

## Generated outputs

When run from the repository root, the notebook creates:

```text
figures/
  fig1_vqe_convergence.png
  fig2_qse_conditioning.png
  fig3_qse_levels.png
  fig4_noise_ground_state.png
  fig5_error_cancellation.png
  fig6_classical_convergence.png
  fig7_hardware.png

results/
  all_results.json
  hardware_job.json
  hardware_results.json
```

Hardware-job retrieval may depend on the submitting IBM Quantum account and the provider's data-retention policy. For long-term reproducibility, archive the derived results and non-sensitive execution metadata in the repository.

## Scope and limitations

- STO-3G and the selected active spaces cannot provide quantitatively converged spectroscopy for the diffuse ethylene $V$ state.
- The comparison uses two fixed geometries, not a torsional potential-energy surface.
- Expectation-value diagnostics do not prove exact particle-number or spin-sector support.
- The FakeManila simulation is representative; it is not a calibration-faithful model of the IBM Fez job.
- Three repeats from one hardware job do not characterize device drift or provide a robust uncertainty estimate.
- The exploratory electron-transfer coupling is not reported because a common-axis vector projection and a fragment-aware localized-state treatment are still required.

## Citation

If you use this work, please cite the report:

```bibtex
@misc{aboushmeila2026ethylene,
  author       = {Abdelrahman A. AbouShmeila},
  title        = {Spin-Resolved VQE-QSE Simulation of Ethylene},
  year         = {2026},
  month        = sep,
  note         = {Alexandria Quantum Hackathon submission}
}
```

## Acknowledgments

This work was developed for the Alexandria Quantum Hackathon 2026 using Qiskit, Qiskit Nature, PySCF, and IBM Quantum resources. The full scientific references are listed in the accompanying report.

