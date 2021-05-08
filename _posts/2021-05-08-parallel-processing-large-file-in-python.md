---
layout: post
title: Parallel processing large file in Python
---

Have you ever wondered how to increase the performance of your program? Applying parallel processing is a powerful method for better performance. In today's post, we are going to solve a problem by applying this method. I'll explain the solution step by step.

## Problem Statement

Let's start with the problem statement. We are given a large text file that weights ~2.4GB and consists of 400,000,000 lines. Our goal is to find the most frequent character for each line.

You can use the following command in your terminal to create the input file:

```bash
yes Hello Python! | head -n 400000000 > input.txt
```

## Line Processor Algorithm

Here you can plug in any algorithm to process a line. For the sake of this tutorial, I am going to stick to the following logic. This function calculates the most frequent letter for a given line:

```python
def process_line(line):
    # Count frequency for every character
    counter = {}
    for letter in line:
        if letter not in counter:
            counter[letter] = 0
        counter[letter] += 1

    # Find the character with the most frequency using `counter`
    most_frequent_letter = None
    max_count = 0
    for key, value in counter.items():
        if value >= max_count:
            max_count = value
            most_frequent_letter = key
    return most_frequent_letter
```

## Serial Processing

Trivial processing: open up the file, read through every line, and pass it to the above algorithm.

```python
def serial_read(file_name):
    results = []
    with open(file_name, 'r') as f:
        for line in f:
            results.append(process_line(line))
    return results
```

## Parallel Processing

This is the interesting part. We need to understand how an efficient solution should look like. This diagram should give you an idea of what's going to happen next.

```
             file_name
             chunk_start
             chunk_end ┌─────────────────────┐chunk_results
             ┌─────────►    process_chunk    ├────────┐
             │         └─────────────────────┘        │
             │                                        │
             │         ┌─────────────────────┐        │
             │     ┌───►    process_chunk    ├─┐      │
 file_name   │     │   └─────────────────────┘ │      │    results
   ┌─────────┴─────┴┐                         ┌▼──────▼───────┐
───►  parallel_read │                         │ parallel_read ├─►
   └─────────┬─────┬┘                         └▲──────▲───────┘
             │     │   ┌─────────────────────┐ │      │
             │     └───►    process_chunk    ├─┘      │
             │         └─────────────────────┘        │
             │                                        │
             │         ┌─────────────────────┐        │
             └─────────►    process_chunk    ├────────┘
                       └─────────────────────┘
```

As you can see, we are going to need a couple of functions: 
- `parallel_read` 
  - Takes `file_name` as input
  - Opens the file
  - Splits the file into smaller chunks. Note that we don't read the entire file when splitting it into chunks. We read some of the lines to figure out where a line starts to avoid breaking the line while splitting into chunks.
  - Delegates the chunks to multiple processes
  - Combines results from different processes into `results` 
  - Returns `results`
- `process_chunk` 
  - Takes `file_name`, `chunk_start` (character position to start processing from), `chunk_end` (character position to end at) as input
  - Opens the file
  - Reads lines from `chunk_start` to `chunk_end`
  - Passes the lines to our main algorithm - `process_line`
  - Stores the result for the current chunk in `chunk_results`
  - Returns `chunk_results`

We could refactor these functions by decoupling the logic but let's keep them as is to understand the higher-level picture.

This is how `parallel_read` looks like in Python:

```python
def parallel_read(file_name):
    # Maximum number of processes we can run at a time
    cpu_count = mp.cpu_count()

    file_size = os.path.getsize(file_name)
    chunk_size = file_size // cpu_count

    # Arguments for each chunk (eg. [('input.txt', 0, 32), ('input.txt', 32, 64)])
    chunk_args = []
    with open(file_name, 'r') as f:
        def is_start_of_line(position):
            if position == 0:
                return True
            # Check whether the previous character is EOL
            f.seek(position - 1)
            return f.read(1) == '\n'

        def get_next_line_position(position):
            # Read the current line till the end
            f.seek(position)
            f.readline()
            # Return a position after reading the line
            return f.tell()

        chunk_start = 0
        # Iterate over all chunks and construct arguments for `process_chunk`
        while chunk_start < file_size:
            chunk_end = min(file_size, chunk_start + chunk_size)

            # Make sure the chunk ends at the beginning of the next line
            while not is_start_of_line(chunk_end):
                chunk_end -= 1

            # Handle the case when a line is too long to fit the chunk size
            if chunk_start == chunk_end:
                chunk_end = get_next_line_position(chunk_end)

            # Save `process_chunk` arguments
            args = (file_name, chunk_start, chunk_end)
            chunk_args.append(args)

            # Move to the next chunk
            chunk_start = chunk_end

    with mp.Pool(cpu_count) as p:
        # Run chunks in parallel
        chunk_results = p.starmap(process_chunk, chunk_args)

    results = []
    # Combine chunk results into `results`
    for chunk_result in chunk_results:
        for result in chunk_result:
            results.append(result)
    return results
```

And here is the implementation of `process_chunk`:

```python
def process_chunk(file_name, chunk_start, chunk_end):
    chunk_results = []
    with open(file_name, 'r') as f:
        # Moving stream position to `chunk_start`
        f.seek(chunk_start)

        # Read and process lines until `chunk_end`
        for line in f:
            chunk_start += len(line)
            if chunk_start > chunk_end:
                break
            chunk_results.append(process_line(line))
    return chunk_results
```

## Performance Measurement

I am going to take the simplest approach to measure the performance by capturing the time before and after execution. Here is what it looks like:

```python
def measure(func, *args):
    time_start = time.time()
    result = func(*args)
    time_end = time.time()
    print(f'{func.__name__}: {time_end - time_start}')
    return result
```

Running both serial and parallel solutions:

```python
if __name__ == '__main__':
    measure(serial_read, 'input.txt')
    measure(parallel_read, 'input.txt')
```

## Results

We can save ~4 minutes with parallel processing:

```
serial_read: 785.301323890686
parallel_read: 547.9388830661774
```

That's a win! Note that the maximum number of processes I could run was 4. You can save more time with a better machine.

## Caveats

Parallel processing is not a one-size-fits-all solution because with smaller data the trivial algorithm will perform a lot better. That's because a trivial algorithm doesn't require additional time for managing multiple processes.

The splitting by chunks algorithm is based on the number of bytes/characters. So you might notice that some chunks process a lot of short lines whereas other chunks process a single long line. I think this approach fits perfectly with the problem we are trying to solve. Feel free to adjust it to your needs.

Let me know if you spot any bugs, learned something new, or would like to improve the solution.

*Complete implementation available in [this GitHub Gist](https://gist.github.com/nurdabolatov/f8eb1f0bee6b4a60b2f00f6b900c73fb)*