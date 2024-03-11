# Week 1 Exercises: Hello world
## Part 1: Getting oriented
Creating Rust Project with [cargo](https://doc.rust-lang.org/cargo/getting-started/installation.html)  
cargo is the package management tool for the Rust programming language. It compiles and generates executable files based on the package list provided by Cargo.toml.
```sh
$ cd week1/part-1-hello-world
$ cargo build
$ ./target/debug/
```
When building a project using `cargo build`, the executable file will be generated in the `target/debug` directory. The `debug` directory represents the default parameters for building, compiling, and generating executable files in debug mode. Another parameter is `release`, where more optimizations are applied during compilation compared to debug mode, resulting in longer compilation times, and the generated files are located in the `target/release` directory.

## Part 2: Rust warmup
Getting familiar with Rust syntax quickly through functions requires understanding the following two points:
* Default immutability of variables in Rust
  * Variables declared in Rust are by default considered immutable types. If a variable is not annotated with `mut`, any attempt to reassign a value to it will result in a compilation error.
  * Example: `let mut a = 0;` must be declared with `mut` to allow reassignment.
* Distinguishing between expressions and statements
  * If a line of code ends with `;`, it represents a statement and does not return a value. Otherwise, it is an expression with a return value.
  * Except for single-line code, a code block `{...}` can also be distinguished based on whether the last line ends with `;`.
  * In summary, if a function has a return value, it should be placed on the last line of the code block without ending with `;`.

### `fn add_n`
Given an integer `n` and a vector `v`, add `n` to each value in the vector `v` and return the result.
```rust
fn add_n(v: Vec<i32>, n: i32) -> Vec<i32> {
    let mut ret: Vec<i32> = Vec::new();
    for num in v.iter() {
        ret.push(*num + n);
    }
    ret
}
```
Since the parameter `v` passed in is immutable, a new vector is declared with `mut` to allow modification. Each element in the vector is accessed via an iterator and added to `n`, then pushed into the new vector `ret`.

Note that dereferencing is necessary for the iterator to be added to the `i32` type `n`.

### `fn add_n_inplace`
Given an integer `n` and a vector `v`, add `n` to each value in the vector `v` without declaring a new vector.
```rust
fn add_n_inplace(v: &mut Vec<i32>, n: i32) {
    for num in v.iter_mut() {
        *num += n;
    }
}
```
Unlike the previous exercise, this time the vector passed in has a `mut` label, indicating that elements within the vector can be modified. Thus, `iter_mut` is used to modify elements in the vector through iteration.

### `dedup`
Given a vector `v`, remove duplicate elements from it in place.
Using a `HashSet` to record numbers that have already appeared, and deleting them from `v` when encountered later.
```rust
fn dedup(v: &mut Vec<i32>) {
    let mut seen: HashSet<i32> = HashSet::new();
    let mut i = 0;
    
    while i < v.len() {
        let num = v[i];
        if seen.contains(&num) {
            v.remove(i);
        } else {
            i += 1;
        }
        seen.insert(num);
    }  
}
```
Since it relies on the index `i` to access vector elements, if `v[i]` is deleted, elements after index `i` will be shifted forward. In this case, `i` does not need to be incremented to access the next element.

## Part 3: Hangman
Implementing the Hangman guessing game in a command-line interface. Hangman is a word-guessing game where players need to guess the given word within a limited number of incorrect guesses. Each turn, a player can guess one letter, and if the letter is in the word, the positions where it appears will be revealed.

At the beginning of the program, it reads words from the file `words.txt`, randomly selects one, sets the number of incorrect guesses to 5, and ends the game when the number of incorrect guesses is reached or the word is guessed correctly.

```rust
let secret_word = pick_a_random_word();
let secret_word_chars: Vec<char> = secret_word.chars().collect();
let mut answer = String::from("-".repeat(secret_word_chars.len()));
let mut cnt = 0;
```
The part for randomly selecting a word is predefined. We declare two variables, `answer` and `cnt`, to track the current guessing status.

Here's the main logic of the game:
```rust
loop {
    let chr;
    let mut input = String::new();
    let mut tmp: Vec<char> = answer.chars().collect();
    let mut flag = 0;

    println!("The  word so far is {}", answer);
    println!("You have guessed the following letters:");
    println!("You have {} guesses left", NUM_INCORRECT_GUESSES - cnt);
    print!("Please guess a letter: ");
    io::stdout().flush().unwrap();
    io::stdin().read_line(&mut input).expect("Failed to read line\n");
    chr = input.chars().nth(0).unwrap();
    

    for i in 0..secret_word_chars.len() {
        tmp[i] = if secret_word_chars[i] == chr {
            flag = 1;
            chr
        } else {
            tmp[i]
        }
    }

    answer = tmp.into_iter().collect();
    if flag == 0 {
        cnt += 1;
        println!("Sorry, that letter is not in the word");
    }
    
    print!("\n\n");

    if answer == secret_word || cnt == NUM_INCORRECT_GUESSES {
        break;
    }
}
```
First, the game status is displayed, including the current guess result and remaining guesses. Since `print!("Please guess a letter: ");` does not end with `'\n'`, it might be buffered in the I/O buffer. Immediately using `flush` ensures the result is displayed promptly.

`stdin` reads user input and stores it in the `string`. Considering that the user may input more than one character, only the first character of the input is compared with `secret_word`. If a match is found, the corresponding character in `tmp` is replaced with the guessed letter.

Based on the comparison result, appropriate messages are displayed to the user, and `answer` is updated for the next loop to show where the correctly guessed letters are located. The loop breaks when the word is guessed correctly or the maximum number of incorrect guesses is reached.

```rust
if answer == secret_word {
    println!("Congratulations you guessed the secret word: {}!", answer);
} else {
    println!("Sorry, you ran out of guesses!");
}
```
Finally, the game result is displayed to the user.