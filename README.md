# DIC: Mathematical Program Compression for Discrete Byte Sequences

**Research Prototype / Technical Preprint**

**Author:** Pranay Wajjala  
**Project:** DIC — Discrete Integral Compression  
**Current prototype:** DIC22  
**Date:** October 2026

### Name
**DIC** stands for **Discrete Integral Compression**:
- **D** = Discrete
- **I** = Integral
- **C** = Compression

The name reflects the project's use of discrete finite differences and inverse discrete integration for exact reconstruction.

---

## Abstract

This work presents DIC, a research direction for lossless compression based on representing digital byte sequences as discrete mathematical objects rather than primarily as symbol-frequency or dictionary structures.

The central hypothesis is that a byte stream can be treated as samples of a discrete function

\[
f(n) = x_n
\]

and that useful compression may arise when a transformed sequence — particularly a finite-difference sequence, recurrence, periodic sequence, modular polynomial, packed integer sequence, or composition of these structures — admits a substantially smaller exact mathematical description than the original byte stream.

DIC progressively evolved from finite-difference experiments into a mathematical program representation framework. The current prototype supports modular polynomial representations, differential-periodic models, recurrence models, differential-recurrence models, packed integer domains, hierarchical repeated regimes, parameterized mathematical programs, interleaved streams, segmentation, and exact RAW fallback.

The framework deliberately excludes Huffman coding, arithmetic/range coding, entropy coding, LZ/LZ77/LZMA-style dictionary matching, and conventional dictionary compression from its mathematical representation layer. A candidate representation is accepted only when its actual canonical serialized representation is smaller than the represented data and exact reconstruction succeeds.

Experiments demonstrate extremely compact representations for selected synthetic mathematical sequences. For example, the current research benchmark represents 4,096-byte quadratic, cubic, and recurrence sequences in approximately 73–94 bytes. On a 1.2 MB structured mixed benchmark, DIC22 produces a 553,410-byte representation (ratio 0.461175) with exact reconstruction.

These results demonstrate the feasibility of mathematical program representation as a lossless compression mechanism for selected structured data. They do **not** establish that DIC is superior to established general-purpose compressors across representative real-world corpora. A principal objective of the ongoing work is therefore to establish the domains in which mathematical program compression provides measurable advantages and to characterize its limitations.

---

## 1. Introduction

Lossless compression is usually approached through models of statistical redundancy, repeated substrings, symbol probabilities, contexts, or combinations of these techniques.

DIC investigates a different question:

> Can redundancy in a digital sequence be represented directly as a mathematical program?

Consider a byte sequence:

\[
x_0,x_1,x_2,\ldots,x_{N-1}
\]

Instead of treating the bytes only as symbols, DIC treats them as samples of a discrete function:

\[
f(n)=x_n
\]

The sequence may have mathematical structure that becomes simpler after transformation.

For example, a polynomial sequence may have a constant finite difference at some order. A modular polynomial can exhibit the same property in a finite byte domain even when ordinary integer differences appear to wrap around.

This motivates the basic DIC search principle:

> Search for the lowest differential or mathematical transformation whose resulting sequence has an exact, compact mathematical description.

The transformation itself is not considered compression. Compression occurs only when the complete mathematical description, including all parameters and serialization overhead, is smaller than the source representation.

---

## 2. Research Hypothesis

The principal hypothesis is:

> Some digital byte sequences contain deterministic mathematical structure whose minimum exact program description can be substantially smaller than their literal representation.

This hypothesis does not imply that all data is mathematically compressible.

In particular:

- random data should generally remain RAW;
- encrypted data should generally remain RAW;
- already compressed data may provide little opportunity;
- a mathematical description with large metadata overhead is not a compression win;
- a heuristic pattern is not accepted unless exact reconstruction is verified.

DIC therefore treats RAW as a first-class mathematical-program outcome rather than forcing every input into a mathematical model.

---

## 3. Mathematical Foundation

### 3.1 Discrete finite differences

For a sequence \(f(n)\), the first forward difference is:

\[
\Delta f(n)=f(n+1)-f(n)
\]

Higher-order differences are recursively defined:

\[
\Delta^2 f(n)=\Delta(\Delta f(n))
\]

and generally:

\[
\Delta^k f(n)=\Delta(\Delta^{k-1}f(n))
\]

A polynomial of degree \(d\) has a constant \(d\)-th finite difference over an appropriate integer domain.

DIC extends this idea into finite modular domains.

---

### 3.2 Modular arithmetic

Bytes naturally form a finite domain:

\[
\mathbb{Z}_{256}
\]

Therefore DIC can define:

\[
\Delta f(n) =
(f(n+1)-f(n)) \bmod 256
\]

This is important because ordinary integer finite differences can appear to lose polynomial structure when values wrap from 255 to 0.

Modular finite differences preserve the exact algebraic structure within the finite domain.

For example, the sequence

\[
f(n)=n^2 \bmod 256
\]

has:

\[
\Delta^2 f(n)=2 \bmod 256
\]

and can therefore be represented by a small modular polynomial program.

---

### 3.3 Newton-style representation

DIC uses finite-difference structure to construct exact polynomial descriptions.

A sequence may be represented using its initial values and finite-difference coefficients rather than enumerating every sample.

This provides a discrete analogue of describing a function through a compact set of mathematical parameters.

---

## 4. Mathematical Program Grammar

The prototype has progressively expanded its exact grammar.

### 4.1 RAW

Literal bytes are stored directly.

RAW is essential because the framework must not claim structure where none exists.

---

### 4.2 Modular polynomial / differential models

A sequence can be represented through a finite difference order and the corresponding modular structure.

The serialized representation contains the parameters necessary for exact reconstruction.

---

### 4.3 Periodic mathematical sequences

A sequence whose transformed representation has period \(P\) can be described by:

\[
g(n)=g(n\bmod P)
\]

rather than storing every sample.

The periodic model is validated across the entire candidate interval.

---

### 4.4 Recurrence models

DIC can represent sequences satisfying finite-order recurrences such as:

\[
x_n = a_1x_{n-1}+a_2x_{n-2}+\cdots+a_rx_{n-r}+b
\pmod M
\]

The initial values and recurrence coefficients are sufficient to reconstruct the sequence.

---

### 4.5 Differential recurrence

A recurrence can itself exist in a finite-difference sequence.

Conceptually:

\[
\Delta^k f(n)=g(n)
\]

where \(g(n)\) satisfies a recurrence.

This composes discrete calculus with recurrence synthesis.

---

### 4.6 Packed integer domains

Some binary files represent integers across multiple bytes.

DIC therefore examines packed 16-, 32-, and 64-bit domains rather than restricting all mathematical analysis to individual bytes.

For example:

\[
x_n \in \mathbb{Z}_{2^{32}}
\]

can expose structure hidden by treating every individual byte independently.

---

### 4.7 Segmentation

A file does not need to obey one mathematical law globally.

DIC therefore searches for representations such as:

```text
MATHEMATICAL REGION
RAW REGION
PERIODIC REGION
MATHEMATICAL REGION
RAW REGION
...
```

Candidate intervals compete using actual serialized byte cost.

---

### 4.8 Hierarchical repetition

DIC can recognize repeated sequences of mathematical regimes.

Instead of describing every regime independently, a higher-level program can describe a repeated arrangement of mathematical programs.

---

### 4.9 Parameterized mathematical programs

The DIC21 generation introduced parameterized mathematical structures.

The underlying idea is that a family of related mathematical regions may be represented by:

\[
P(\theta_n)
\]

where the parameters \(\theta_n\) themselves follow a mathematical rule.

This creates a hierarchy:

\[
\text{data}
\rightarrow
\text{program}
\rightarrow
\text{parameters}
\rightarrow
\text{parameter program}
\]

The objective is to reduce the description cost of repeated mathematical regimes.

---

### 4.10 Interleaved streams

DIC also investigates cases where mathematical structure exists across multiple interleaved streams.

For example, a byte sequence may be separated into subsequences:

\[
x_0,x_2,x_4,\ldots
\]

and

\[
x_1,x_3,x_5,\ldots
\]

where each stream has a simpler mathematical description than the combined sequence.

---

## 5. Exact Reconstruction

Losslessness is a mandatory property.

A candidate model is not accepted merely because it appears mathematically plausible.

The pipeline is:

```text
Input
  ↓
Candidate mathematical programs
  ↓
Actual canonical serialization
  ↓
Decode
  ↓
Byte-for-byte equality
  ↓
SHA-256 validation
```

A model that fails exact reconstruction is rejected.

This also prevents discovery heuristics from becoming claims of compression.

---

## 6. L'Hospital-Inspired Discovery

The project investigated whether concepts related to limits and L'Hospital's rule could assist mathematical discovery.

L'Hospital's rule itself is not directly applicable to finite byte sequences because the data is discrete and finite.

Instead, DIC uses the concept only as inspiration for discovery heuristics.

For example, successive finite-difference ratios can provide clues about possible differential order or asymptotic behavior.

These heuristics are never sufficient for acceptance.

Exact modular verification and actual serialized cost remain authoritative.

---

## 7. Serialization Principle

A mathematical pattern is not automatically a compression result.

Suppose a 1,000-byte sequence can be described by a mathematical equation, but the complete serialized program requires 1,200 bytes.

That is not compression.

DIC therefore evaluates:

\[
C_{\text{program}} < C_{\text{source}}
\]

rather than:

\[
\text{mathematical elegance}
\]

or:

\[
\text{estimated entropy}
\]

The canonical serialized byte stream is the final authority.

---

## 8. Experimental Results

### 8.1 Synthetic mathematical benchmark

The current DIC22 research suite contains deterministic datasets designed to test specific mathematical structures.

Representative results:

| Dataset | Input | DIC22 | Ratio |
|---|---:|---:|---:|
| cubic | 4,096 B | 79 B | 0.019287 |
| quadratic | 4,096 B | 78 B | 0.019043 |
| recurrence1 | 4,096 B | 73 B | 0.017822 |
| recurrence2 | 4,096 B | 75 B | 0.018311 |
| diff_recurrence1 | 4,096 B | 84 B | 0.020508 |
| packed32_linear | 4,096 B | 86 B | 0.020996 |
| poly_periodic | 4,096 B | 94 B | 0.022949 |
| piecewise_affine | 4,096 B | 140 B | 0.034180 |
| repeated_records | 4,096 B | 131 B | 0.031982 |
| random | 4,096 B | 4,165 B | 1.016846 |

All tested datasets reconstruct exactly.

The random-data result is important: DIC does not currently demonstrate useful mathematical compression for deterministic random data and appropriately falls back to RAW, with container overhead.

---

### 8.2 Mixed structured benchmark

A 1,200,000-byte deterministic mixed benchmark was used to test whether mathematical and RAW regions can coexist.

DIC produced:

- Input: 1,200,000 bytes
- Canonical representation: 553,410 bytes
- Ratio: 0.461175
- Mathematical coverage: 655,203 bytes
- RAW coverage: 544,797 bytes
- Segments: 15
- Exact reconstruction: YES

The representation contained mathematical periodic regions, hierarchical repeated regimes, and RAW intervals.

The SHA-256 of the reconstructed output matched the original.

---

### 8.3 Comparison with established compressors

On the same mixed benchmark, one recorded experiment produced approximately:

| Compressor | Output |
|---|---:|
| DIC | 553,410 B |
| gzip -9 | 550,975 B |
| xz -9 | 553,480 B |
| zstd -19 | 546,172 B |

These results are **benchmark-specific**.

They do not establish general superiority.

The experiment demonstrates that a mathematical-program representation can approach established compressors on at least one deliberately structured mixed corpus, while using a fundamentally different representation strategy.

---

## 9. Computational Cost

Compression speed is currently a significant limitation.

For the 1.2 MB mixed benchmark, the DIC21/DIC22-generation implementation required approximately 11 seconds for analysis/encoding in the recorded experiments.

The same benchmark was compressed much faster by conventional compressors.

This is expected from the current architecture because DIC performs expensive mathematical model discovery and global candidate selection.

Therefore, DIC currently trades computational complexity for mathematical program discovery.

Future work must address:

- multiscale candidate discovery;
- caching;
- incremental differential computation;
- early rejection of impossible models;
- parallel candidate evaluation;
- more efficient dynamic programming;
- model-cost lower bounds;
- reusable mathematical fingerprints.

---

## 10. What DIC Does Not Claim

This project does **not** currently claim:

1. universal superiority over gzip, xz, zstd, Brotli, or other compressors;
2. information-theoretic compression beyond the encoded program;
3. compression of arbitrary random or encrypted data;
4. that every file possesses a compact mathematical representation;
5. that synthetic mathematical benchmarks represent general-world workloads;
6. that DIC's current implementation is production-ready;
7. that the current prototype has undergone independent peer review.

The correct current claim is narrower:

> DIC demonstrates an exact mathematical-program representation framework capable of compressing selected structured digital sequences by representing their underlying mathematical structure rather than relying on conventional entropy or dictionary coding.

---

## 11. Research Questions

### Q1. Is differentiation itself compression?

No.

Differentiation is reversible only when the required initial conditions are retained, and the transformed representation still requires serialization.

Compression occurs only when the complete mathematical description is smaller.

### Q2. Can modular arithmetic recover structure hidden by byte wrapping?

Yes.

Experiments demonstrate exact modular polynomial representations for quadratic and cubic byte sequences.

### Q3. Can multiple mathematical laws coexist in one file?

Yes.

Global segmentation permits mathematical and RAW regions to coexist.

### Q4. Can a mathematical program contain another mathematical program?

This is a central direction of the DIC21/DIC22 research.

Parameterized mathematical programs allow program parameters to be described structurally rather than independently.

### Q5. What happens when mathematical discovery fails?

The sequence remains RAW.

### Q6. Does DIC currently beat general-purpose compressors?

Not generally established.

The mixed synthetic benchmark is close to gzip and xz and remains larger than zstd in the recorded comparison. Broader representative corpora are required.

---

## 12. Limitations

The current implementation has several important limitations.

### 12.1 Search complexity

The mathematical grammar creates a potentially large candidate space.

Finding the globally cheapest mathematical program can become computationally expensive.

### 12.2 Grammar incompleteness

A sequence may contain a compact mathematical structure that is not expressible in the current grammar.

Failure to compress therefore does not imply absence of mathematical structure.

### 12.3 Metadata overhead

Short mathematical regions can be more expensive to describe than their literal bytes.

### 12.4 Real-world validation

The current strongest results are on deliberately constructed mathematical datasets and one structured mixed benchmark.

A larger independent corpus is required.

### 12.5 Speed

Current encoding speed is substantially behind mature general-purpose compressors.

---

## 13. Future Work

The planned research directions include:

### 13.1 Multiscale mathematical discovery

Search simultaneously at multiple granularities:

```text
whole file
    ↓
large blocks
    ↓
medium blocks
    ↓
small blocks
    ↓
interleaved streams
```

### 13.2 Parameter-program recursion

Allow mathematical parameters to themselves be generated by mathematical programs.

### 13.3 Program algebra

Introduce compositions such as:

\[
P(Q(n))
\]

and:

\[
P(n) + Q(n)
\]

where exact serialization remains the acceptance criterion.

### 13.4 Interleaved mathematical structures

Search for multiple independent mathematical streams within binary data.

### 13.5 Better packed domains

Investigate additional word sizes, signed representations, endianness, and structured fields.

### 13.6 Global minimum-description search

Develop stronger lower bounds and pruning strategies for mathematical program selection.

### 13.7 Independent benchmarking

Evaluate DIC on representative public corpora containing:

- source code;
- text;
- CSV;
- JSON;
- logs;
- binary executables;
- scientific data;
- databases;
- network captures;
- images;
- audio;
- archives;
- already compressed data;
- encrypted/random data.

---

## 14. Reproducibility

The project is intended to be reproducible.

The repository contains:

- the DIC implementation;
- deterministic research datasets;
- benchmark commands;
- exact encode/decode validation;
- SHA-256 validation;
- research notes.

Typical commands:

```bash
python3 dic22_math_program_engine.py selftest

python3 dic22_math_program_engine.py generate-tests dic22_tests

python3 dic22_math_program_engine.py benchmark dic22_tests

python3 dic22_math_program_engine.py analyze dic5_benchmark/mixed.bin

python3 dic22_math_program_engine.py encode \
    dic5_benchmark/mixed.bin \
    mixed.d22

python3 dic22_math_program_engine.py decode \
    mixed.d22 \
    mixed_restored.bin

cmp dic5_benchmark/mixed.bin mixed_restored.bin
```

For external comparison:

```bash
python3 dic22_math_program_engine.py market-corpus \
    dic22_corpus \
    --json dic22_corpus_results.json
```

---

## 15. Conclusion

DIC explores lossless compression from a different starting point: not primarily "How can repeated symbols or substrings be encoded more efficiently?", but:

> **"What mathematical program exactly generates this digital sequence, and is that program smaller than the sequence itself?"**

The experiments demonstrate that this approach can represent selected polynomial, recurrence, periodic, packed-integer, piecewise, hierarchical, and parameterized structures using very small exact descriptions.

The mixed benchmark demonstrates that mathematical and literal regions can coexist in a single canonical representation.

However, the current evidence is not sufficient to claim that DIC is a replacement for established general-purpose compression algorithms. The remaining challenge is to determine whether mathematical-program compression provides a reproducible advantage across meaningful real-world data classes, while reducing its current computational cost.

DIC is therefore best viewed at its current stage as an **open research prototype and experimental compression framework**.

The goal of the project is not to prove the original hypothesis by assumption.

The goal is to test it rigorously.

---

## 16. Availability

Source code, deterministic test data, benchmark tooling, and research documentation are intended to be published through the project's GitHub repository.

Repository:

**[INSERT GITHUB REPOSITORY URL]**

Research paper:

**[INSERT GITHUB RESEARCH_PAPER.md URL]**

---

## 17. Citation

If this work is useful in research or derivative software, please cite the corresponding repository release and research paper.

A `CITATION.cff` file should be added to the repository for machine-readable citation metadata. GitHub supports `CITATION.cff` and exposes a "Cite this repository" interface when it is present.

---

## 18. License

The source code and research artifacts should be released under an explicit open-source license.

The repository should state the license before public release.

---

## Appendix A — Design Principle

The central design principle can be summarized as:

\[
\boxed{
\text{compression} =
\text{exact mathematical program}
+
\text{canonical serialization}
}
\]

subject to:

\[
C_{\text{program}} < C_{\text{literal}}
\]

and:

\[
D(E(x)) = x
\]

for every accepted input \(x\).

If these conditions are not satisfied, the mathematical representation is not accepted as a compression result.

---

## Appendix B — Terminology

**Discrete function:** A sequence interpreted as samples \(f(n)\).

**Finite difference:** A discrete analogue of differentiation.

**Modular difference:** A finite difference calculated within a modular domain.

**Mathematical program:** A compact exact description capable of reconstructing a sequence.

**Mathematical coverage:** Input bytes represented by non-RAW mathematical models.

**RAW coverage:** Input bytes represented literally.

**Canonical model size:** The actual serialized size of the DIC representation.

**Program selection:** Selection among mathematically valid representations according to actual serialized cost.

**Hierarchical program:** A program containing other mathematical programs or repeated mathematical regimes.

**Parameterized program:** A mathematical program whose parameters vary according to another structured rule.

---

## Appendix C — Reproducibility Statement

All numerical claims in this document should be treated as prototype measurements tied to the stated datasets, implementation version, and execution environment.

Benchmark results should be regenerated after significant implementation changes.

No benchmark number should be interpreted as a universal property of DIC without independent replication.
