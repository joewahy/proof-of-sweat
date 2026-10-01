# ProofOfSweat

Zero-knowledge proofs of fitness achievements, built from scratch in Python.

Prove "I ran 5k in under 25 minutes" without revealing your GPS route, your exact splits, or your workout log.

> **Status: not yet started.** This README describes the design and plan. No protocol code exists yet. Sections marked *(planned)* are goals, not features.

## Motivation

In 2018, Strava's public heatmap exposed the locations of secret military bases, because aggregated route data revealed where people were running. The achievement ("I ran this far") was never the sensitive part. The data that made it true was.

ProofOfSweat explores the alternative: let someone prove a fitness claim while the underlying data never leaves their device.

## How it works *(planned)*

The claim is a threshold statement: *my committed value is at most T* (for example, finish time ≤ 25:00).

1. **Pedersen commitment.** The prover commits to their value `v` as `C = v·G + r·H`. The commitment hides `v` (hiding) and cannot later be opened to a different value (binding).
2. **Sigma protocol.** A Schnorr-style proof of knowledge: the prover shows they know the secret behind a group element without revealing it.
3. **Fiat-Shamir transform.** The verifier's random challenge is replaced by a hash of the transcript, making proofs non-interactive and serializable.
4. **Bit-decomposition range proof.** To show `T - v ≥ 0`, the prover decomposes `T - v` into bits, commits to each bit, and proves each is 0 or 1 with an OR-proof (simulating the branch they do not know). The bit commitments must recombine to the commitment of `T - v`.
5. **Composed claim.** `prove_achievement` and `verify_achievement` wrap the above into one API: the verifier learns that `value ≤ threshold` and nothing else about `value`.

## Roadmap

| # | Milestone | Goal |
|---|-----------|------|
| 0 | Setup | Project folder, venv, dependencies, `.gitignore` |
| 1 | Foundations | EC group wrapper, Pedersen `commit` / `open`, binding tests |
| 2 | Sigma protocol | Interactive Schnorr proof of knowledge of a discrete log |
| 3 | Fiat-Shamir | Non-interactive, serializable Schnorr proof |
| 4 | Range proof | Bit-decomposition with OR-proofs (hardest step) |
| 5 | Real claim | Prove `threshold - value` is in range; clean prover/verifier API |
| 6 | Application | CLI to prove from a workout stat, verifier CLI or FastAPI endpoint, optional mini leaderboard |
| 7 | Docs | Full protocol write-up and security properties |

Stretch goals: signed or attested input data, and Bulletproofs for smaller proofs.

## Security properties *(target)*

- **Completeness:** an honest prover with a true claim always convinces the verifier.
- **Soundness:** a prover whose claim is false cannot convince the verifier, except with negligible probability.
- **Zero-knowledge:** the verifier learns nothing beyond the truth of the claim. Proofs can be simulated without the secret.

## Limitations

A zero-knowledge proof shows that a committed value satisfies the claim. It does **not** show that the value came from a real workout. Nothing here stops a prover from committing to a fabricated number.

Closing that gap requires tying the commitment to attested data from a trusted source, such as signed wearable or GPS data. That is out of scope and listed as future work. It is not solved by this project.

This is also an educational implementation. It has not been audited and should not be used where real security matters.

## Tech stack

- Python 3.11+
- `py_ecc` or `coincurve` for elliptic curve operations (existing library for the EC math, with the protocol layers written from scratch)
- `pytest`
- CLI first, with an optional small FastAPI verifier endpoint later

## License

Not yet chosen.
