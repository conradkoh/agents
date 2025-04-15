# Eval
The term eval has its own nuance in the context of AI. This aims to tackle eval on a more abstract level.
## Definitions

### Eval
An eval is a function that takes an input and returns an output.

### Process
A process consists of an eval and a set of inputs and corresponding outputs.

###  Context
The context of an process is the intermediate outputs of each application of the eval function.

## Linear Evals
A linear eval is an eval that is a process where the input is either the original input or the output of the previous step.

### Single Step Linear Eval
A single step linear eval is an eval that is a minimally linear process whose function is only applied once.
> Let $x$ be the input to the eval </br>
> Let $y$ be the output of the eval </br>
> An eval is a function $y=eval(x)$

### Two Step Linear Eval
A two step linear eval is an eval that is a minimally linear process whose function is applied twice.
> Let $x$ be the input to the eval </br>
> Let $y$ be the output of the eval </br>
> An eval is a function $y=eval(eval(x))$

### Multi Step Linear Eval
A multi step linear eval is an eval that is a minimally linear process whose function is applied multiple times.
> Let $x$ be the input to the eval </br>
> Let $y$ be the output of the eval </br>
> An eval is a function $y=eval(eval(...eval(x)))$

##  Contextual Multi-Step Evals
### Simple Context
Simple context is the simple concatenation of all preceding outputs as context to the next application of the eval function.

### Distance Adjusted Context
Distance adjusted context is a context that adjusts the contribution of each preceding output based on the distance from the current step.

## Types of Context driven Linear Evals
### Simple Context Linear Evals
A simple context linear eval is a linear eval that is a process where the eval takes in the original input and the output of all the previous steps.
> Let $x$ be the input to the eval </br>
> Let $y$ be the output of the eval </br>
> An eval is a function $y=eval(x,eval(x),eval(eval(x)),...)$

```typescript
class SimpleContext {
  state: string[];
  constructor() {
    this.state = [];
  }
  add(x: string) {
    this.state.push(x);
  }

  async get(): Promise<string> {
    return this.state.join("\n");
  }
}
/**
 * Simple context linear eval
 * @param x - The input to the eval
 * @param context - The context of the eval
 * @returns the output of the eval
 */
function simpleContextEval(x: any, context: SimpleContext) {
  if(terminationCondition(x, context)) {
    return x;
  }
  return simpleContextEval(x, context);
}
/**
 * @param x - The input to the eval
 * @param context - The context of the eval
 * @returns true if the eval should terminate, false otherwise
 */
function terminationCondition(x: any, ...context: any[]): boolean {
  //TODO: Implement termination condition
}
```

### Linear Evals With Compression
A linear eval with compression is a linear eval that augments the significance of each intermediate output based on the distance from the current step.

```typescript
type ContextItem = { 
  data: string, //current context item
  tokenCount: number, 
  rawData: string, //original user input
  compressed?: {
    low?: string,
    medium?: string,
    high: string //we always generate as high a level of summary as possible
  }
}
type ContextState = { items: ContextItem[], tokenCount: number };
//CompressionStrategy is a function that helps to compress key
type CompressionStrategy = {
  compress: (items: ContextItem[], currentIdx: number) => Promise<ContextItem[]>;
  compressionTokenThreshold: number;
}
class ContextWithCompression {
  state: ContextState;
  constructor(compressionStrategy: CompressionStrategy) {
    this.state = { items: [], tokenCount: 0 };
    this._compressionStrategy = compressionStrategy;
  }

  /**
   * Get the context items into a single string
   * @param items - The context items
   * @param currentIdx - The current index of the item
   * @returns the summarized context
   **/
  async get(): Promise<string> {
    if(this.state.tokenCount > this._compressionTokenThreshold) {
      //a compression is in order
      this.compress();
    }
    return this.state.items.map(item => item.data).join("\n");
  }
  /**
   * Compress the context
   */
  private async compress() {
    const currentIdx = this.state.items.length - 1;
    const compressedItems = await this._compressionStrategy.compress(this.state.items, currentIdx);
    this.state.items = compressedItems;
  }

  /**
   * Adds an item to the context
   * @param item - The item to add
   */
  add(val: string) {
    this.state.push(item);
  }
}

/**
 * Linear eval with compression
 * @param x - The input to the eval
 * @param context - The context of the eval
 * @returns the output of the eval
 */
function evalWithCompression(x: any, ...context: ContextWithCompression[]) {
  if(terminationCondition(x, ...context)) {
    return x;
  }
  return evalWithCompression(x, ...context);
}

/**
 * @param x - The input to the eval
 * @param context - The context of the eval
 * @returns true if the eval should terminate, false otherwise
 */
function terminationCondition(x: any, ...context: ContextWithCompression[]): boolean {
  //TODO: Implement termination condition
}

```

## Distance Based Compression
A distance based compression is a compression that augments the significance of each intermediate output based on the distance from the current step.

```typescript
class DistanceBasedCompression implements CompressionStrategy {
  compressionTokenThreshold: number;
  constructor(compressionTokenThreshold: number) {
    this.compressionTokenThreshold = compressionTokenThreshold;
  }
  compress(items: ContextItem[], currentIdx: number): Promise<ContextItem[]> {
    type AnnotatedItem = ContextItem & { position: number };
    const annotateItemsWithPosition = items.map((item, index) => ({
      ...item,
      position: index
    }));
    const segments: AnnotatedItem[][] = segmentItemsByDistance(annotateItemsWithPosition, currentIdx);
    const compressedItems = segments.map(segment => {
  }
  
}
```
