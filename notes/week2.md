# Week2
## `read_file_lines`
```rust
fn read_file_lines(filename: &String) -> Result<Vec<String>, io::Error> {
    let mut ret: Vec<String> = Vec::new();
    let file = File::open(filename)?;    
    for line in io::BufReader::new(file).lines() {
        let line_str = line?;
        ret.push(line_str)
    }
    Ok(ret)
}
```
* If `let file = File::open(filename)?;` or `let line_str = line?;`, then `io::Error` will be `return`

## Implementing the Grid interface
The starter code use 1D vector to store a grid, so we need to a method to convert 2D coordinates to 1D coordinates.
```rust
fn convert_2d_to_1d(&self, row: usize, col: usize) -> usize {
    row * self.num_cols + col
}    
```
After implementing the coordinate transformation method, the next step is to implement the `get` and `set` functions.
```rust
pub fn get(&self, row: usize, col: usize) -> Option<usize> {
    if (self.is_valid_index(row, col)) {
        let index = self.convert_2d_to_1d(row, col);
        Some(self.elems[index])
    } else {
        None
    }
}
pub fn set(&mut self, row: usize, col: usize, val: usize) -> Result<(), &'static str> {
    if (self.is_valid_index(row, col)) {
        let index = self.convert_2d_to_1d(row, col);
        self.elems[index] = val;
        Ok(())
    } else {
        Err("Out of range\n")
    }
}
```
* We need to ensure that the incoming coordinates do not exceed the bounds
* The return value needs to be decorated according to the types defined by `Result` and `Option`
  * `Option<usize>` requires returning `None` and `Some(usize)`
  * `Result<(), &'static str>` requires returning `Ok(())` and `Err(&'static str)`

## Longest Common Subsequence (LCS)
[LCS](https://en.wikipedia.org/wiki/Longest_common_subsequence) is a classical dynamic programming algorithm. Its execution involves:
ts execution involves:

1. Initialization: Set up a table to store intermediate results, typically based on the lengths of input sequences.

1. Iterative Calculation: Traverse the table, calculating the length of the longest common subsequence at each cell.

1. Update DP Table: Fill the table based on the calculated lengths.

1. Backtracking: Reconstruct the longest common subsequence by backtracking through the table.

1. Output: The longest common subsequence found is the final result.

The starter code splits the above procedure into two parts: in the `fn lcs` function, it calculates the DP table, while backtracking and output are handled in the `fn print_diff` function.
```rust
fn lcs(seq1: &Vec<String>, seq2: &Vec<String>) -> Grid {
    let m = seq1.len();
    let n = seq2.len();
    let mut dp = Grid::new(m+1, n+1);
    for i in 0..m {
        for j in 0..n{
            let mut val;
            if seq1[i] == seq2[j] {
                val = dp.get(i, j).unwrap()+1;
            } else {
                val = max(dp.get(i+1, j).unwrap(), dp.get(i, j+1).unwrap());
            }
            dp.set(i+1, j+1, val).unwrap();
        }
    }
    dp
}
```
* Use `unwrap` retrive values from `Result` and `Option`
* Add `use std::cmp::max;` at the top of the `main.rs` file to import `max` for usage

## `print_diff`
When LCS table is calculated, we need to use this to find difference between two sequences.
```rust
fn print_diff(lcs_table: &Grid, lines1: &Vec<String>, lines2: &Vec<String>, i: usize, j: usize) {
    if i > 0 && j > 0 && lines1[i-1] == lines2[j-1] {
        print_diff(lcs_table, lines1, lines2, i-1, j-1);
        println!("  {}", lines1[i-1]);
    } else if j > 0 && (i == 0 || lcs_table.get(i, j-1).unwrap() >= lcs_table.get(i-1, j).unwrap()) {
        print_diff(lcs_table, lines1, lines2, i, j-1);
        println!("> {}", lines2[j-1]);
    } else if i > 0 && (j == 0 || lcs_table.get(i, j-1).unwrap() < lcs_table.get(i-1, j).unwrap()) {
        print_diff(lcs_table, lines1, lines2, i-1, j);
        println!("< {}", lines1[i-1]);
    } else {
        println!("");
    }
}
```
* This function backtracks `lcs_table` to find updated path.
  * `lines1[i-1] == lines2[j-1]` means the current value is updated from top-left corner
  * Otherwise, it finds the larger value between the top and the left