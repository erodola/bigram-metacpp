# Compile-Time Bigram Language Model

This project pushes the boundaries of C++ template metaprogramming by implementing a **Language Model that performs inference entirely at compile-time**. 

Instead of calculating text during runtime, the compiler itself acts as the inference engine, baking the final generated string directly into the binary's data segment.

## Why?

Hardware isn't always the bottleneck; our definition of when execution should happen is. 

We currently use compilers to optimize model paths, but we can also use them to resolve deterministic parts of the model itself. This project is a proof-of-concept for "shipping code that already knows the answer."

## Features

* **`constexpr` Inference Engine**: A complete bigram Markov chain implementation that runs during the compilation phase.
* **Compile-Time RNG**: Since compilers are deterministic, this uses a hashed combination of `__TIME__` and `__DATE__` to seed a `constexpr` Xorshift32 generator.
* **Zero Runtime Overhead**: The generated text is stored as a static constant. The runtime CPU cost for generating the text is exactly zero cycles.

## How it works

The project implements a character-level bigram model. The transition probabilities (trained externally) are encoded into a `static constexpr` matrix.

1. **Seeding**: The preprocessor macros `__TIME__` and `__DATE__` are hashed using an FNV-1a algorithm to create a unique seed for each build.
2. **Sampling**: An implementation of [Inverse Transform Sampling](https://en.wikipedia.org/wiki/Inverse_transform_sampling) picks the next token based on the current context's probability distribution.
3. **Baking**: The `NameGenerator` struct iterates through the Markov chain until a termination character (`.`) is sampled or the max length is reached.
4. **Result**: The final string is embedded in the binary. You can verify this by inspecting the binary's strings or assembly output (I use [ImHex](https://github.com/WerWolv/ImHex)).

## Code preview

It's just one cpp file. Here's how you generate names:

```cpp
// Inference happens here. If you wait one second and recompile, 
// the binary will contain a different name.

static constexpr NameGenerator<15> result(seed, T);

int main() {
    // Zero computation happens here. It just prints a constant string.
    std::cout << "Generated Name: " << result.name << std::endl;
}

```

Each time the cpp is compiled, a new name is generated.

## Compatibility

Tested with the MSVC (`cl.exe`) compiler only. 

It relies on standard C++17 `constexpr` and template features, so it should theoretically work with GCC and Clang as well, provided they support the necessary recursion depth for complex template instantiations.

---

*Useless? Absolutely. But a reminder that the C++ compiler is Turing complete.*
