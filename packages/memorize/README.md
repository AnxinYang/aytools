# `withMemorized`

![Test](https://github.com/AnxinYang/aytools/actions/workflows/test.yml/badge.svg)

A TypeScript utility function that wraps a given function with a caching layer, allowing you to memorize function results with optional time-to-live (TTL) and maximum cache size limits. This is particularly useful for optimizing expensive function calls by avoiding redundant computations.

## Features

- **Time-to-Live (TTL)**: Set a time limit on how long a result is cached before it's considered stale.
- **Maximum Cache Size**: Limit the number of results stored in the cache. Oldest results are evicted when the cache size exceeds this limit.
- **Memory Control**: Import, export, and clear cache memory as needed.

## Installation

To install the package, you can use npm or yarn:

```bash
npm install @aytools/memorize
```

or

```bash
yarn add @aytools/memorize
```

## Usage

### Basic Usage

```typescript
import { withMemorized } from '@aytools/memorize';

async function expensiveCalculation(x: number): Promise<number> {
  // Simulate an expensive calculation
  return x * 2;
}

const [memorizedCalculation] = withMemorized(expensiveCalculation, {
  ttl: 5000,       // Cache results for 5 seconds
  maxCacheSize: 3  // Limit cache size to 3 results
});

// Use the memorized function
const result = await memorizedCalculation('calc-1', 10);
console.log(result);  // Outputs: 20
```

### Memory Control

You can manage the cache using the provided control functions:

```typescript
const [memorizedCalculation, { importMemory, exportMemory, clearMemory }] = withMemorized(expensiveCalculation);

// Import a memory object
importMemory({
  'calc-1': { value: 20, expiration: Date.now() + 5000 }
});

// Export the current memory object
const memory = exportMemory();
console.log(memory);

// Clear the cache
clearMemory();
```

## API

### `withMemorized`

```typescript
function withMemorized<FunctionType extends InputFunctionType>(
  fn: FunctionType,
  options?: MemorizeOptions
): [MemorizedFunction<FunctionType>, MemoryControlFunctions<FunctionType>];
```

#### Parameters

- **`fn`**: The function to be memorized.
- **`options`**: (Optional) An object of type `MemorizeOptions` containing the following fields:
  - **`ttl`**: `number` (optional) - Time-to-live in milliseconds for a cached result. If not provided, results will be cached indefinitely.
  - **`maxCacheSize`**: `number` (optional) - Maximum number of results to store in the cache. If the cache exceeds this size, the oldest result will be removed.

#### Returns

A tuple consisting of:

1. **`memorizedFunction`**: The function wrapped with the caching layer.
2. **`MemoryControlFunctions`**: An object with the following methods:
   - **`importMemory(memory: Record<string, MemorizedResult<ReturnType<FunctionType>>>)`**: Imports a memory object into the current cache.
   - **`exportMemory(): Record<string, MemorizedResult<ReturnType<FunctionType>>>`**: Exports the current memory object.
   - **`clearMemory(): void`**: Clears all cached results.

## Example

Here is an example of how you might use the `withMemorized` utility in a real-world scenario:

```typescript
import { withMemorized } from '@aytools/memorize';

async function fetchDataFromAPI(endpoint: string): Promise<any> {
  const response = await fetch(endpoint);
  return response.json();
}

const [cachedFetch] = withMemorized(fetchDataFromAPI, { ttl: 10000, maxCacheSize: 5 });

async function getData() {
  const data = await cachedFetch('user-data', 'https://api.example.com/user/123');
  console.log(data);
}
```

In this example, `fetchDataFromAPI` results will be cached for 10 seconds, and the cache will store up to 5 results before evicting the oldest ones.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request or open an issue to discuss improvements or bug fixes.

## License

This project is licensed under the MIT License.