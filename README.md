# Triplix
[![PyPI version](https://badge.fury.io/py/triplix.svg)](https://badge.fury.io/py/triplix)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/triplix.svg)
[![MIT](https://img.shields.io/pypi/l/triplix.svg?color=green)](https://opensource.org/licenses/MIT)

A python package to efficiently process, store and retrieve spatial genomics multi-contact data.

## Introduction:
Triplix provides access to multi-contact data, that could be stored in three different spaces (i.e., resolution): 
1. **Concatemers**: An HDF5 container that stores multi-contact data in a table-like format. 
Each row in this table is an alignment that is captured by a particular `concatemer` (i.e., a sequenced long `read`). 
There is an index (i.e., `read_idx`) column in this table that specifies alignments that originate from each specific `concatemer`.

2. **Tri-alignments**:
Every possible three cis-alignments of each `concatemer` is stored in the Tri-alignment file. Each three cis-alignment is called a `tri-alignment`. 
Such a data structure facilitates 3-way interaction analyses, and allow random-access. 
To this end, `Triplix` has a simple wrapper around [pairix](https://github.com/4dn-dcic/pairix).
In particular, the relevant `tri-alignments` are loaded by checking the first two given coordinates using `pairix`, and then filtering for the third coordinate via an ordinary overlap check.

3. **Triplets**:
Triples are essentially Tri-alignment files, where `tri-alignments` are aggregated into genomic bins. 
This not only reduces the disk (and memory) footprint, but also facilitates random-access. Considering the sparcity of the multi-contact data, most (if not all) multi-contact analyses are done in this "binned" space (rather than `alignment` space).

## Installation
You may run the following commands in your command line to install Triplix.
```bash
pip3 install triplix

# or
python3 -m pip install triplix
```

## Usage:
Below, you can see how you can extract data in each space.

#### 1. Concatemer space  (*.concatemers.h5)
Provides random-access to read (i.e., `concatemer`) level data. 

First, we open `concatemers` container via:
```python
import triplix

concatemers_path = './concatemers/mESC.MC-HiC.concatemers.h5'
concatemers_container = triplix.ConcatemersContainer(file_path=concatemers_path)
```

Then, we can access the content, by requesting a specific concatemer/read index, or range of concatemer indices:
```python
print(concatemers_container[6])
# <Concatemers>
#    read_idx  chrom_num     start       end  strand  map_quality  flag  seq_start  seq_end                             read_name  read_length
# 0         6          9  28049720  28051533      -1           60  2064         22     1833  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 1         6          9  27993951  27996370       1           60     0       2233     4617  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 2         6          9  28001381  28003048      -1           60  2064       4602     6245  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 3         6          9  28003034  28003230      -1           60  2064       6429     6621  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 4         6          9  34934155  34935542       1           60  2048       6615     7929  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 5         6          9  36134828  36135055      -1           42  2064       7925     8149  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 6         6         15  48738095  48738498      -1           60  2064       8145     8536  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 7         7          2  63927407  63930186       1           60  2048         23     2803  000a76b2-216e-4c40-b0f5-35c7ce10d30c         8151
# 8         7          2  64210922  64221347       1           60     0       2793     8134  000a76b2-216e-4c40-b0f5-35c7ce10d30c         8151


print(concatemers_container[6:9])
# <Concatemers>
#     read_idx  chrom_num      start        end  strand  map_quality  flag  seq_start  seq_end                             read_name  read_length
# 0          6          9   28049720   28051533      -1           60  2064         22     1833  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 1          6          9   27993951   27996370       1           60     0       2233     4617  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 2          6          9   28001381   28003048      -1           60  2064       4602     6245  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 3          6          9   28003034   28003230      -1           60  2064       6429     6621  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 4          6          9   34934155   34935542       1           60  2048       6615     7929  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 5          6          9   36134828   36135055      -1           42  2064       7925     8149  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 6          6         15   48738095   48738498      -1           60  2064       8145     8536  000a9e3e-5a06-4691-acf7-f71c0b707bc9         8538
# 7          7          2   63927407   63930186       1           60  2048         23     2803  000a76b2-216e-4c40-b0f5-35c7ce10d30c         8151
# 8          7          2   64210922   64221347       1           60     0       2793     8134  000a76b2-216e-4c40-b0f5-35c7ce10d30c         8151
# 9          8         10  103377945  103378320       1           60  2048         25      391  000a93a0-5d61-42a1-81e2-848b7b197f60         3710
# 10         8         10  105640948  105642721      -1           60    16        387     2134  000a93a0-5d61-42a1-81e2-848b7b197f60         3710
# 11         8         15   43523758   43523837      -1           38  2064       2130     2209  000a93a0-5d61-42a1-81e2-848b7b197f60         3710
# 12         8         13    4296589    4296895       1           60  2048       2205     2501  000a93a0-5d61-42a1-81e2-848b7b197f60         3710
# 13         8         19   48817930   48819166       1            1  2048       2497     3708  000a93a0-5d61-42a1-81e2-848b7b197f60         3710
```

Alternatively, one can iterate over all `concatemers`:
```python
import pandas as pd
concatemers_iter = concatemers_container.iter(return_name=True, return_length=True)
n_concatemer = len(concatemers_container)
for idx, concatemer in enumerate(concatemers_iter):
    df = pd.DataFrame(concatemer)
    print(df[['chrom_num', 'start', 'strand', 'read_name']])
    if idx > 0:
        break

#    chrom_num     start  strand                             read_name
# 0          6  19951796       1  0000b7ad-14d7-4285-a1a3-65a87eef213d
# 1          6  18571279       1  0000b7ad-14d7-4285-a1a3-65a87eef213d
# 2          6  19944712      -1  0000b7ad-14d7-4285-a1a3-65a87eef213d
# 3          6  18954688      -1  0000b7ad-14d7-4285-a1a3-65a87eef213d
# 4          6  18958868       1  0000b7ad-14d7-4285-a1a3-65a87eef213d
# 5          6  20007053      -1  0000b7ad-14d7-4285-a1a3-65a87eef213d
# 6          6  19139306       1  0000b7ad-14d7-4285-a1a3-65a87eef213d
# 7         11  74436453      -1  0000b7ad-14d7-4285-a1a3-65a87eef213d
#    chrom_num     start  strand                             read_name
# 0          5  92933191      -1  0000eea6-4554-40c3-b4ab-ecaa5cf5795b
# 1          5  23899224       1  0000eea6-4554-40c3-b4ab-ecaa5cf5795b
# 2          5  33962641      -1  0000eea6-4554-40c3-b4ab-ecaa5cf5795b
# 
```

#### 2. Tri-alignment space (*.tri-alignments.tsv.bgz):
Provides access to alignment (i.e., `tri-alignments`) level data. 
These files are essentially formatted as compressed [pairix files](https://github.com/4dn-dcic/pairix). The difference is that they contain three coordinates instead of two coordinates (as is the case for regular pairix files)

> **Note:** The tri-alignment files need to be indexed if you like to have random access to them. You can do this using `triplix index --input=input-file.tri-alignments.tsv.bgz`

An example of this usage is as follows:

```python
import triplix

tri_aln_path = './tri-alignments/GM12878.Merged.tri-alignments.tsv.bgz'
tri_container = triplix.TriAlignmentsContainer(trialn_path=tri_aln_path)

tri_alignments_iter = tri_container.fetch(chrom_a='chrY', start_a=0e6, end_a=10e6, batch_size=10)
for idx, tri_alignments in enumerate(tri_alignments_iter):
    df = tri_alignments.to_dataframe()
    print(df[['read_name', 'chrom_A', 'start_A', 'start_B', 'start_C']])
    if idx > 0:
        break
#                               read_name chrom_A  start_A  start_B  start_C
# 0  b9129419-3e9d-47ea-8ae3-bf65c468ef43    chrY    10710   363575   363575
# 1  b9129419-3e9d-47ea-8ae3-bf65c468ef43    chrY    10710   363575   449218
# 2  b9129419-3e9d-47ea-8ae3-bf65c468ef43    chrY    10710   363575   449218
# 3  689000cd-3deb-47f9-8c8d-64eb039d0be1    chrY    10710  1203147  1302472
# 4                  SRR11589401.15490287    chrY    11129  1774864  1806803
# 5                  SRR11589391.12389645    chrY    13063   493484  1644842
# 6  998d7719-0d64-4202-ba7e-0a4b9d49284e    chrY    13337   310549   672618
# 7  998d7719-0d64-4202-ba7e-0a4b9d49284e    chrY    13337   310549   754824
# 8  998d7719-0d64-4202-ba7e-0a4b9d49284e    chrY    13337   672618   754824
#                               read_name chrom_A  start_A  start_B  start_C
# 0  e7603585-2e15-423e-a26f-8b114d6bab87    chrY    13338    17122   266818
# 1                  SRR11589390.11433849    chrY    13394    13813  2515291
# 2  e7f93e28-9540-462b-817c-d8acd563b94a    chrY    13394    13844   318989
# 3                   SRR11589394.4086145    chrY    13441  1231515  2705073
# 4  429af33b-3260-4dc5-9cf3-fc7b96f0b932    chrY    13508   252336   323450
# 5  0f526116-e8fc-459b-aac9-d9d6d2843289    chrY    13584   278514   364567
# 6  0f526116-e8fc-459b-aac9-d9d6d2843289    chrY    13584   278514   366665
# 7  6589f1d9-cd7d-4cba-bfc9-06f123956157    chrY    13584   282655   440783
# 8  0f526116-e8fc-459b-aac9-d9d6d2843289    chrY    13584   364567   366665
# 9  11f46754-fbfe-4f9f-8aa6-7f03c7db48df    chrY    13584  1180971  1812799

```

#### 3. Triplet space (*.triplets.h5):
Provides access to triplet-level data. A usage example is:

```python
import pandas as pd
import triplix

trpl_path = './triplets/GM12878.MC-HiC.hg38.triplets.h5'
trpl_obj = triplix.TripletsContainer(container_path=trpl_path)
triplets_input = trpl_obj.fetch(
    chrom_a='chr8', start_a=119e6, end_a=120e6,
    columns=['start_A', 'start_B', 'start_C', 'count_ABC'],
)
df = pd.DataFrame(triplets_input)
print(df)
# output:
#           start_A    start_B    start_C  count_ABC
# 0       119000000  119000000  119000000         73
# 1       119000000  119000000  119025000        136
# 2       119000000  119000000  119050000         82
# 3       119000000  119000000  119075000         55
# 4       119000000  119000000  119100000         54
# ...           ...        ...        ...        ...
# 136156  120000000  121950000  121975000          8
# 136157  120000000  121950000  122000000          2
# 136158  120000000  121975000  121975000          5
# 136159  120000000  121975000  122000000          7
# 136160  120000000  122000000  122000000          6
# 
# [136161 rows x 4 columns]
```

