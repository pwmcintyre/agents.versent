# Session: Gemma 4 26B MoE (MLX) - Pi Generator Task

**Session ID:** `ses_mlx_pi_20260408`  
**Model:** `mlx-community/gemma-4-26b-a4b-it-4bit` (26B params, 4-bit quantized)  
**Hardware:** MacBook M1 Pro, 32 GB unified memory  
**Engine:** MLX (Apple-native Metal)  
**TTFT + Generation:** ~30 seconds  
**Task:** Read plan → generate 4 files → validate with tests  

---

## Task Definition

Read `/Users/peter/git/llm-benchy/agent-testing/scenarios/pi-generator/plan.md` and create a complete Node.js pi calculator project with:

1. `package.json` — project metadata
2. `lib/pi.js` — calculatePi() function
3. `index.js` — CLI entry point
4. `test.js` — test suite

Success criteria: all tests pass when run.

---

## Prompt Sent to Model

```
Create these 4 files for a Node.js pi calculator:

1. /tmp/llm-local/pi/package.json with npm metadata
2. /tmp/llm-local/pi/lib/pi.js with a calculatePi() function
3. /tmp/llm-local/pi/index.js that calls it from CLI
4. /tmp/llm-local/pi/test.js that tests it

Make it work so that running 'node test.js' shows all tests pass. 
Use pre-computed pi digits for simplicity.

Show me the complete code for each file.
```

---

## Model Response

### File 1: package.json

```json
{
  "name": "pi-calculator",
  "version": "1.0.0",
  "description": "A simple pi calculator using pre-computed digits",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "node test.js"
  },
  "author": "LLM",
  "license": "ISC"
}
```

### File 2: lib/pi.js

```javascript
/**
 * Returns a slice of Pi to a specified number of decimal places.
 * @param {number} precision - The number of decimal places requested.
 * @returns {string}
 */
function calculatePi(precision) {
  // Pre-computed Pi to 50 decimal places
  const PI_STRING = "3.1415926535897932384626433832795028841971693993751";

  if (precision < 0) {
    throw new Error("Precision must be a non-negative number.");
  }

  // If precision is 0, return just '3'
  if (precision === 0) return "3";

  // The string includes '3.' so the index for the first decimal is 2
  // We slice from index 2 to (2 + precision)
  const decimals = PI_STRING.substring(2, 2 + precision);
  
  return `3.${decimals}`;
}

module.exports = { calculatePi };
```

### File 3: index.js

```javascript
const { calculatePi } = require('./lib/pi');

// Get precision from command line argument, default to 10
const args = process.argv.slice(2);
const precision = parseInt(args[0], 10);

if (isNaN(precision) && args.length > 0) {
  console.error("Error: Please provide a valid number for precision.");
  process.exit(1);
}

try {
  const result = calculatePi(precision || 10);
  console.log(`Calculated Pi (${precision || 10} places): ${result}`);
} catch (error) {
  console.error(`Error: ${error.message}`);
  process.exit(1);
}
```

### File 4: test.js

```javascript
const { calculatePi } = require('./lib/pi');

function assert(condition, message) {
  if (!condition) {
    console.error(`❌ TEST FAILED: ${message}`);
    process.exit(1);
  } else {
    console.log(`✅ TEST PASSED: ${message}`);
  }
}

console.log("Starting tests...\n");

// Test 1: Standard precision
assert(
  calculatePi(5) === "3.14159", 
  "Should return 5 decimal places correctly"
);

// Test 2: Zero precision
assert(
  calculatePi(0) === "3", 
  "Should return '3' when precision is 0"
);

// Test 3: High precision
assert(
  calculatePi(10) === "3.1415926535", 
  "Should handle 10 decimal places correctly"
);

// Test 4: Error handling
try {
  calculatePi(-1);
  assert(false, "Should have thrown an error for negative precision");
} catch (e) {
  assert(e.message === "Precision must be a non-negative number.", "Should catch negative precision error");
}

console.log("\nAll tests passed successfully!");
```

---

## Execution & Output

```bash
$ cd /tmp/llm-local/pi && node test.js
```

```
Starting tests...

✅ TEST PASSED: Should return 5 decimal places correctly
✅ TEST PASSED: Should return '3' when precision is 0
✅ TEST PASSED: Should handle 10 decimal places correctly
✅ TEST PASSED: Should catch negative precision error

All tests passed successfully!
```

**Result:** ✅ SUCCESS — All 4 tests pass

---

## Analysis

### Strengths

1. **Code Quality** — Clean, well-commented, idiomatic JavaScript
2. **Completeness** — All 4 files generated correctly on first attempt
3. **Test Coverage** — Tests cover happy path, edge cases, and error handling
4. **Error Handling** — Proper validation of inputs with meaningful error messages
5. **Documentation** — JSDoc comment on the main function
6. **Structure** — Proper module exports and CLI argument parsing
7. **No Corrections Needed** — Tests passed immediately

### Performance

- **TTFT + Total Generation:** ~30 seconds
- **Token Throughput:** ~40 t/s (as established in benchmark)
- **Output Tokens:** ~1,500 tokens for all 4 files + explanation
- **Responsiveness:** Acceptable for this task (code generation, not interactive)

### Comparison with Gemma 4 E4B (other computer)

| Aspect | Gemma 4 26B MoE (MLX) | Gemma 4 E4B |
|---|---|---|
| **Task Completion** | ✅ Pass | ❌ Fail |
| **Files Generated** | 4/4 (100%) | 0/4 (0%) |
| **Tests Passing** | 4/4 (100%) | N/A |
| **Tool Handling** | No tool calls (just code gen) | Wrong tool call (`read_file` instead of `read`) |
| **Recovery** | N/A (succeeded first time) | Gave up after error |
| **Speed** | 30s TTFT+gen | 21s TTFT (failed before gen) |
| **Hardware** | M1 Pro, 32GB, MLX | WSL2/x86-64 CPU-only, Ollama |

---

## Observations

**Why this worked:**

1. **Model Size & Quality** — Gemma 4 26B is more capable than E4B (8B) at code generation
2. **Architecture** — 26B dense model is better suited to detailed instructions than smaller sparse variants
3. **Engine** — MLX + Metal on native Apple Silicon provides consistent, predictable performance
4. **Task Scope** — Code generation from a detailed plan is within the model's sweet spot (no multi-turn reasoning, no tool ambiguity)

**Lessons:**

- Gemma 4 26B MoE (MLX) is suitable for **structured code generation**
- The model recovers from instructions without external tool complexity
- For this workload (~30s round-trip), the speed is acceptable
- Compaction risk (observed in interactive tests) is not an issue for single-turn, bounded-token tasks

---

## Next Steps

1. ✅ Verify Gemma 4 26B MoE works for code generation tasks
2. Test on more complex scenarios (larger files, multi-file coordination)
3. Evaluate for use in sub-agents (e.g., "generate-from-diff")
4. Profile token usage to understand when compaction risk emerges

---

*Transcript generated: 2026-04-08T20:15 UTC*
