# Issue

**Title:** Speed up exponentials of complex numbers.

## Description

## Description
This PR replaces two calls of `exp` of a complex number, where the real part is also zero, with Euler's formula.

## Motivation and Context
There are two places in freud where a complex number is passed into `exp` with the real part being hard coded to zero. One case is in  `StaticStructureFactorDirect`, identified in #1506, but we decided to make a separate PR. Searching for the pattern`"std::exp(std::complex<float>(0"` showed the same thing occurring in `Hexatic`. 

I'm not totally familiar with all of the c++ implementation details, but using `exp` on a complex number ultimately uses `cexpf` which contains a few extra processes that we can skip with Euler's formula if the real part is always zero.  

## How Has This Been Tested?

I tested the time performance with the below, just using one density and k_max value while varying `N`.

```python
import time

import freud

def time_ssf(N, density=1.0, k_max=5.0):
    L = (N / density) ** (1 / 3)
    box, points = freud.data.make_random_system(L, N, seed=0)
    sf = freud.diffraction.StaticStructureFactorDirect(bins=100, k_max=k_max)
    t0 = time.perf_counter()
    sf.compute((box, points))
    return time.perf_counter() - t0
```

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/114334b1-5a2d-4a71-9071-e3ddc6c6aade" />


| N | main (s) | PR (s) | Speedup |
|---:|---:|---:|---:|
| 1,000 | 0.0058 | 0.0028 | 2.10x |
| 6,000 | 0.1357 | 0.0393 | 3.45x |
| 11,000 | 0.4411 | 0.1427 | 3.09x |
| 16,000 | 0.9579 | 0.3149 | 3.04x |
| 21,000 | 1.6293 | 0.5020 | 3.25x |
| 26,000 | 2.5117 | 0.8020 | 3.13x |
| 31,000 | 3.5277 | 1.1122 | 3.17x |

## Checklist:
- [x] I have reviewed the [**Contributor Guidelines**](https://github.com/glotzerlab/freud/blob/main/CONTRIBUTING.md).
- [x] I agree with the terms of the [**Freud Contributor Agreement**](https://github.com/glotzerlab/freud/blob/main/ContributorAgreement.md).
- [x] My name is on the [list of contributors](https://github.com/glotzerlab/freud/blob/main/doc/source/reference/credits.rst) in the pull request source branch.
- [x] I have updated the [Change log](https://github.com/glotzerlab/freud/blob/main/ChangeLog.md).

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `be563f126675`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.