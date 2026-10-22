# Lean proof of K₂ ≥ 0.301657

This directory contains a proof in Lean 4 of the bound K₂ ≥ 0.301657 for the Bohr radius of the bidisc, under one
hypothesis, the inequality A₃ + B₂²/2 ≤ 2 (Lemma 3.4 of the paper). Section 7 of the paper describes the proof.

The main theorem is in `Bohr59aN/Final.lean`:

    theorem Bohr59aN.K2_ge_N (hPC : PatelColligation) : K2Claim

- `K2Claim` is `(301657 / 1000000 : ℝ) ≤ K2` (`Bohr59a/Statement.lean`). There `K2` is the supremum of the radii r > 0
  at which Bohr's inequality holds for every family of coefficients whose power series converges absolutely on the
  bidisc to values of modulus less than 1 (`BohrClass`, `BohrIneq`).
- `PatelColligation` (`Bohr59a/Hypotheses.lean`) states A₃ + B₂²/2 ≤ 2 for the coefficients of every function H on
  the bidisc with H(0) = 1 and positive real part. Its proof in the paper uses Andô's inequality and Agler's
  realization, which are not in Mathlib.

## Axioms

`lake env lean AxN.lean` prints the axioms of the main theorem; `AXIOMS.txt` is the expected output. They are the three
standard axioms `propext`, `Classical.choice` and `Quot.sound`, and 49 auxiliary axioms
`Bohr59aCheck.<NAME>.ok._native.native_decide.ax_1_1`, one per part of the certificates: `NAME` is `K5` for the S₅
certificate and `K6S00`, ..., `K6S47` for the 48 shards of the S₆ certificate. Each comes
from a theorem `ok : checkShard shard = true` in `Bohr59aCheck/<NAME>.lean`, proved by `native_decide`, which runs the
compiled checking program. So the trusted base includes the Lean compiler, the C compiler that it uses, the Lean
runtime and the compiled core-library functions that the checks call.

The checking program `checkShard` (`Bohr59aCore/Check.lean`, with the ball arithmetic of `C64Core/Ival.lean`) is
written in core Lean and contains no definitions marked `implemented_by`, `extern`, `unsafe`, `opaque` or `partial`.
Every theorem of the proof that does not use the 49 checks depends only on the three standard axioms. Among them are
the correctness proof of the checking program (`Bohr59a/CertShard.lean`, `CertVertex.lean`, `CertGeom.lean` and
`C64/Ival/`), `Bohr59a.K2_ge_patel` (`Bohr59a/FinalPatel.lean`), which proves Patel's bound K₂ ≥ 0.30158758 under the
same hypothesis, and `Bohr59a.K2_ge_of_sidon` (`Bohr59a/FinalV3.lean`), which derives `K2Claim` from the hypothesis
and the bounds S₅ ≤ 2.187 and S₆ ≤ 2.363.

## Files

- `Bohr59a/`: the definitions, the analytic part of the proof and the correctness proof of the checking program.
- `Bohr59aCore/`: the checking program (`Check.lean`), the data format (`Format.lean`) and the decoder for the
  certificate data (`Decode.lean`), compiled to native code.
- `C64Core/Ival.lean` and `C64/Ival/`: ball arithmetic and its correctness proof.
- `Bohr59aCheck/`: the 49 checks, one module per part, written by `tools/gen_checks.py`.
- `Bohr59aN/Final.lean`: the main theorem, written by `tools/gen_final_n.py`.
- `stats/`: per-part counts and data checksums (`<NAME>.json`), a summary (`DATA_SUMMARY.txt`), and the time and
  memory that each check took in the reference build (`N_RUN_v2.txt`).
- `tools/`: `export59.py` writes the data modules from the certificates; `gen_checks.py` and `gen_final_n.py` write
  the check modules and `Bohr59aN/Final.lean`.
- `data_SHA256SUMS`: the SHA-256 checksums of the 49 data modules `Bohr59aData/<NAME>.lean`.
- `AxN.lean`, `AXIOMS.txt`: the axiom query and its expected output.
- `lakefile.toml`, `lake-manifest.json`, `lean-toolchain`: the project configuration.

The 49 data modules `Bohr59aData/<NAME>.lean` (1.5 GB) are not included. Each holds one part's data as a base64
string.

## Generating the data modules

Unpack `s6_certificate.tar.gz` into the folder `certificate/` of the repository, as described in the main README. Then,
in this directory, run

    CERT=../certificate
    mkdir -p v3/bb6
    ln -s "$(cd "$CERT" && pwd)/s5_certificate.json" v3/bb5_t2187.json
    for s in $(seq 0 47); do ln -s "$(cd "$CERT" && pwd)/s6_certificate/sh$s.json" v3/bb6/sh$s.json; done
    python3 -I -B tools/export59.py k5 v3 .
    python3 -I -B tools/export59.py k6 v3 . $(seq 0 47)
    sha256sum -c data_SHA256SUMS

The exporter reads the file names `bb5_t2187.json` and `bb6/sh<s>.json`, hence the links. It needs only Python 3,
takes about six minutes on one core, and rewrites `stats/<NAME>.json` with the same content. The data modules need not
be trusted: the header of each part is fixed in its check module, and the checking program validates the cells and
every vertex certificate it reads.

## Building

Install `elan`, the Lean toolchain manager. `lean-toolchain` selects Lean `v4.35.0-rc3`, and `lake-manifest.json` pins
Mathlib at commit `c55e6e786f49471c72fbddbec5415808896aec1e` (tag `v4.35.0-rc3`). Then run

    lake exe cache get
    lake build Bohr59aN.Final
    lake env lean AxN.lean

`lake exe cache get` should download compiled Mathlib files; without them, `lake build` compiles Mathlib from source,
which takes much longer. Plain `lake build` does not work here: its default target needs the file `Bohr59a.lean`, which
is not part of the proof and not included. Each of the 49 check modules runs one check. In the reference build they took
between 643 and 1404 seconds and at most about 1 GB of memory each, about 14 CPU-hours in total (`stats/N_RUN_v2.txt`),
and the other modules took minutes. The output of the last command should agree with `AXIOMS.txt`.
