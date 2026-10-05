# Parallel Huffman Compressor in C — Sequential vs. Fork vs. Pthreads

A lossless text compressor and decompressor in C, based on **Huffman coding**, implemented in three concurrency models so their performance can be compared on Linux. Developed for the *Operating Systems* course at the Costa Rica Institute of Technology (TEC), 2025.

## Overview

The program takes a directory of `.txt` files and compresses all of them into a single binary archive. The archive stores a shared Huffman code table and each file's encoded bitstream. The decompressor rebuilds the original files from that archive.

The same algorithm is implemented three ways:

| Version | Concurrency model | How work is split |
|---------|-------------------|-------------------|
| `huffman_compressor` | Sequential | One process encodes every file in turn |
| `huffman_compressor_fork` | Multi-process | One child process per file (`fork()`); results return to the parent through `pipe()` |
| `huffman_compressor_pthread` | Multi-threaded | One POSIX thread per file (`pthread_create`) |

Each compressor and decompressor reports its **total execution time in milliseconds**, so the three approaches can be benchmarked directly against each other.

## How It Works

1. **Read** all `.txt` files in the input directory (up to 100 files).
2. **Count character frequencies** across every file.
3. **Build the Huffman tree** with a min-heap and generate a prefix-free code for each symbol.
4. **Encode** each file into a packed bitstream. This step runs sequentially, in child processes, or in threads, depending on the version.
5. **Write the archive**, which contains:
   - a header,
   - the code table,
   - and for each file: its name, encoded length, byte count and padding bits, followed by the binary data.
6. **Decompress** by reading the code table back and decoding each bitstream into the original file.

## Project Structure

```
P1/
├── huffman_compressor.c           # Sequential compressor
├── huffman_decompressor.c         # Sequential decompressor
├── huffman_compressor_fork.c      # Multi-process compressor (fork + pipes)
├── huffman_decompressor_fork.c    # Multi-process decompressor
├── huffman_compressor_pthread.c   # Multi-threaded compressor (pthreads)
├── huffman_decompressor_pthread.c # Multi-threaded decompressor
├── Makefile
├── install_dependencies.sh        # Installs build-essential and builds everything
├── textos/                        # Sample input .txt files
└── output/                        # Decompressed output
```

## Build

**Requirements:** Linux, GCC and Make.

```bash
cd P1
./install_dependencies.sh   # installs build-essential and runs make
# or, if GCC is already installed:
make all
```

To remove the binaries, run `make clean`.

## Usage

**Compress** a directory of `.txt` files into one archive:

```bash
./huffman_compressor          textos compressed_files.bin
./huffman_compressor_fork     textos compressed_files.bin
./huffman_compressor_pthread  textos compressed_files.bin
```

**Decompress** the archive into a directory (it is created if it doesn't exist):

```bash
./huffman_decompressor          compressed_files.bin output
./huffman_decompressor_fork     compressed_files.bin output
./huffman_decompressor_pthread  compressed_files.bin output
```

**Compare performance** by running all three versions on the same input:

```bash
for v in "" _fork _pthread; do
  echo "== huffman_compressor$v"
  ./huffman_compressor$v textos out$v.bin | grep "ms"
done
```

## Concepts Demonstrated

- Process creation and synchronization: `fork()`, `waitpid()`, `_exit()`
- Inter-process communication with anonymous pipes
- POSIX threads and shared data structures
- Directory traversal and binary file I/O (`dirent.h`, `fread`/`fwrite`)
- Greedy algorithms and priority queues (min-heap Huffman tree)
- Bit-level packing and unpacking
- Timing with `gettimeofday()`

## Tech Stack

C · GCC · POSIX (fork, pipes, pthreads) · Make · Linux

## Authors

Mariano Mayorga Halabi — [GitHub](https://github.com/mdjmayorga) · Computer Engineering, TEC
