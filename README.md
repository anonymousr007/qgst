# Quantum Gate Set Tomography


- Progress in qubit technology requires accurate and reliable methods for qubit characterization.
- There are several reasons characterization is important. One is diagnostics.
- Qubit operations are susceptible to various types of errors due to imperfect control pulses, qubit-qubit couplings (crosstalk), and environmental noise.
- In order to improve qubit performance, it is necessary to identify the types and magnitudes of these errors and reduce them.
- Another reason is the desirability to have metrics of gate quality that are both platform independent and provide sufficient descriptive power to enable the assessment of qubit performance under a variety of conditions. Metrics are also necessary to ascertain if the requirements of quantum error correction (QEC) are being met, i.e., whether the gate error rates are below a suitable QEC threshold.
- Full characterization of quantum processes provides detailed information on gate errors as well as metrics of qubit quality such as gate fidelity.
- Several methods of qubit characterization are currently available.
- In chronological order of their development, the main techniques are quantum state tomography (QST) [1], quantum process tomography (QPT) [2, 3], randomized benchmarking (RB) [4, 5, 6, 7], and quantum gate set tomography (GST) [8, 9]. All of these tools, with the exception of GST, have been well studied and systematized, and have gained widespread acceptance and use in the quantum computing research community.
- GST grew out of QPT, but is somewhat more demanding in terms of the number of experiments required as well as the post-processing. As we will see, obtaining a GST estimate involves solving a highly nonlinear optimization problem.
- In addition, the scaling with system size is polynomially worse than QPT because of the need to characterize multiple gates at once.
- Approximately 80 experiments are required for a single qubit and over 4,000 for 2 qubits to estimate a complete gate set, compared to 16 and 256 experiments respectively to reconstruct a single 1- or 2-qubit gate with QPT.
- Methods for streamlining the resource requirements for GST are under investigation [10,11]. Nevertheless, single-qubit GST has been demonstrated by several groups [8, 9] and 2-qubit GST is widely believed to be achievable.
- Importantly, these groups have convincingly shown that GST outperforms QPT in situations relevant to fault-tolerant quantum information processing (QIP) [8, 9]
- As a result, it seems clear that GST in either its present or some future form will supercede QPT as the only accurate method available to fully characterize qubits for fault-tolerant quantum computing.
- In this document, we present GST in the hopes that readers will be inspired to implement it with their qubits and explore the possibilities it offers.
- GST arose from the observation that QPT is inaccurate in the presence of state preparation and measurement (SPAM) errors.
- It will be useful to classify SPAM errors into two different types, which we will call intrinsic and extrinsic. 
- Intrinsic SPAM errors are those that are inherent in the state preparation and measurement process.
- One example is an error initializing the |0i state due to thermal populations of excited states.
- Another is dark counts when attempting to measure, say, the |1> state.
- Extrinsic SPAM errors are those due to errors in the gates used to transform the initial state to the starting state (or set of states) for the experiment to be performed.
- In QPT, the starting states must form an informationally complete basis of the Hilbert-Schmidt space on which the gate being estimated acts. These are typically created by applying gates to a given initial state, usually the |0> state, and these gates themselves may be faulty.
- Intrinsic SPAM errors are of particular relevance to fault-tolerant quantum computing, since it turns out that QEC requirements are much more stringent on gates than on SPAM.
- According to recent results from IBM [10], a 50-fold increase in intrinsic SPAM error reduces the surface code threshold by only a factor of 3-4. Therefore QPT - the accuracy of which degrades with increasing SPAM - would not be able to determine if a qubit meets threshold requirements when the ratio of intrinsic SPAM to gate error is large.
- This is not an issue for extrinsic SPAM errors, which go to zero as the errors on the gates go to zero.
- Nevertheless, extrinisic SPAM error interferes with diagnostics: as an example, QPT cannot distinguish an over-rotation error on a single gate from the same error on all gates (see Sec. 4.4.1).
- In addition, Merkel, et al. have found that, for a broad range of gate error – including the thresholds of leading QEC code candidates – the ratio of QPT estimation error to gate error increases as the gate error itself decreases [8].
- This makes QPT less reliable as gate quality improves.
- Extrinsic SPAM error is also unsatisfactory from a theoretical point of view: QPT assumes the ability to perfectly prepare a complete set of states and measurements.
- In reality, these states and measurements are prepared using the same faulty gates that QPT attempts to characterize. One would like to have a characterization technique that takes account of SPAM gates self-consistently.
- We shall see that GST is able to resolve all of these issues.
- Another approach to dealing with SPAM errors is provided by randomized benchmarking [4, 5, 6, 7].
- RB is based on the idea of twirling [12, 13] – the gate being characterized is averaged in a such a way that the resulting process is depolarizing with the same average fidelity as the original gate.
- The depolarizing parameter of the averaged process is measured experimentally, and the result is related back to the average fidelity of the original gate.
- This technique is independent of the particular starting state of the experiment, and therefore is not affected by SPAM errors.
- However, RB has several shortcomings which make it unsatisfactory as a sole characterization technique for fault-tolerant QIP.
- For one thing, it is limited to Clifford gates (For recent work discussing generalizations of RB to non-Clifford gates, see Refs. [14, 15].), and so cannot be used to characterize a universal gate set for quantum computing.
- For another, RB provides only a single metric of gate quality, the average fidelity (Recently, generalizations of RB have been proposed to measure leakage rates [16] and the coherence [17] of errors. These advances have the promise to turn RB into a more widely applicable characterization tool.).
- This can be insufficient for determining the correct qubit error model to use for evaluating compatibility with QEC.
- Several groups have shown that qualitatively different errors can produce the same average gate fidelity, and in the case of coherent errors the depolarizing channel inferred from the RB gate fidelity underestimates the effect of the error [18, 19, 20].
- Finally, RB assumes the errors on subsequent gates are independent.
- This assumption fails in the presence of non-Markovian, or time-dependent noise.
- GST suffers from this assumption as well, but the long sequences used in RB make this a more pressing issue.
- Despite these apparent shortcomings, RB has been used with great success by several groups to measure gate fidelities and to diagnose and correct errors [21, 22, 23, 24].
- RB also has the advantage of scalability – the resources required to implement RB (number of experiments, processing time) scale polynomially with the number of qubits being characterized [6, 7].
- QPT and GST, on the other hand, scale exponentially with the number of qubits.
- As a result, these techniques will foreseeably be limited to addressing no more than 2-3 qubits at a time. In our view, GST and RB will end up complementing each other as elements of a larger characterization protocol for any future multi-qubit quantum computer.
- We will not discuss RB further in this document. This document reviews GST and provides instructions and examples for workers who would like to implement GST to characterize their qubits.
- The goal is to provide a guide that is both practical and self-contained.
- For simplicity, consideration is restricted to a single qubit throughout. We begin in Chapter 2 with a review of the mathematical background common to all characterization techniques.
- This includes the representation of gates as quantum maps via the process matrix and Pauli transfer matrix, the superoperator formalism, and the Choi-Jamiolkowski representation, which allows the maximum likelihood estimation (MLE) problem to be formulated as a semidefinite program (SDP).
- Chapter 3 begins with a review of quantum state and process tomography, the characterization techniques that underlie GST.
- We then continue to GST itself, including a derivation of linear-inversion gate set tomography (LGST) following Ref. [9].
- Although LGST is weaker than MLE since it does not in general provide physical estimates, it is useful both as a starting point for the nonlinear optimization associated with MLE, and also in its own right as a technique for getting some information about the gate set quickly and with little numerical effort. This is followed by a description of the MLE problem in both the process matrix [25] and Pauli transfer matrix [26] formulations.
- MLE is the standard approach for obtaining physical gate estimates from tomographic data.
- In Chapter 4 we continue with GST, presenting the detailed experimental protocol as well as numerical results implementing the ideas of Chapter 3.
- The implementation utilizes simulated data with simplified but realistic errors. This data was designed to incorporate the important properties of actual data including coherent and incoherent errors and finite sampling noise.
- Using this simulated data, we compare the performance of QPT and GST.



















