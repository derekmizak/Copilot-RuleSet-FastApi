# GitHub Copilot: Python Performance Optimization

Guidelines for optimizing performance in Python 3.12+ applications, including concurrency patterns, caching strategies, and memory optimization.

**Keywords**: #python #performance #optimization #concurrency #caching #memory

---

## ⚡ Code Performance

* **Algorithm Selection**:
  * Choose appropriate data structures for operations (dict for lookups, list for ordered data)
  * Use set operations for membership testing and deduplication
  * Understand time and space complexity of your algorithms
  * Consider batching operations when processing large datasets
  * Use generators for memory-efficient data processing

* **Python-Specific Optimizations**:
  * Leverage built-in functions (`any`, `all`, `map`, `filter`)
  * Use list/dict/set comprehensions instead of loops when appropriate
  * Utilize standard library modules like `collections`, `itertools`, and `functools`
  * Use `__slots__` for memory-intensive classes
  * Consider native modules for CPU-intensive operations

* **Async Programming**:
  * Use `asyncio` for I/O-bound operations
  * Use `concurrent.futures` for CPU-bound tasks
  * Leverage `async`/`await` syntax for readable asynchronous code
  * Consider `asyncio.gather()` for parallel task execution
  * Use `uvloop` as a faster event loop implementation when appropriate

* **Profiling During Development**:
  * Use simple timing with `time.time()` for quick measurements
  * Use the `timeit` module for benchmarking code snippets
  * Apply `cProfile` for identifying bottlenecks
  * Consider line-by-line profiling with `line_profiler` for critical sections
  * Focus optimization efforts on proven bottlenecks

---

## 🔥 Advanced Performance Patterns

* **Concurrency Patterns**:
  * Use appropriate concurrency model for the task:

  ```python
  import asyncio
  import concurrent.futures
  import time
  from typing import List, Callable, Any

  # CPU-bound task example
  def cpu_bound_task(n: int) -> int:
      """Calculate nth Fibonacci number using a CPU-intensive approach."""
      if n <= 1:
          return n
      return cpu_bound_task(n - 1) + cpu_bound_task(n - 2)

  # IO-bound task example
  async def io_bound_task(delay: float) -> str:
      """Simulate an IO-bound task like a network request."""
      await asyncio.sleep(delay)  # Non-blocking sleep
      return f"Task completed after {delay} seconds"

  # Process CPU-bound tasks with ProcessPoolExecutor
  def process_cpu_bound_tasks(numbers: List[int]) -> List[int]:
      """Process CPU-bound tasks in parallel using multiple processes."""
      with concurrent.futures.ProcessPoolExecutor() as executor:
          results = list(executor.map(cpu_bound_task, numbers))
      return results

  # Process IO-bound tasks with asyncio
  async def process_io_bound_tasks(delays: List[float]) -> List[str]:
      """Process IO-bound tasks concurrently using asyncio."""
      tasks = [io_bound_task(delay) for delay in delays]
      return await asyncio.gather(*tasks)

  # Hybrid approach for mixed workloads
  async def process_mixed_workload(
      cpu_tasks: List[int],
      io_delays: List[float]
  ) -> tuple[List[int], List[str]]:
      """Handle both CPU and IO bound tasks efficiently."""
      # Run CPU-bound tasks in a separate thread pool to avoid blocking the event loop
      loop = asyncio.get_running_loop()
      with concurrent.futures.ProcessPoolExecutor() as pool:
          cpu_task = loop.run_in_executor(
              pool, 
              lambda: process_cpu_bound_tasks(cpu_tasks)
          )
          
          # Concurrently run IO-bound tasks directly with asyncio
          io_results = await process_io_bound_tasks(io_delays)
          
          # Wait for CPU tasks to complete
          cpu_results = await cpu_task
          
      return cpu_results, io_results

  # Usage example
  async def main():
      start = time.time()
      cpu_results, io_results = await process_mixed_workload(
          [30, 31, 32, 33],  # Fibonacci sequence calculations
          [1.0, 1.5, 2.0, 0.5]  # Network request delays
      )
      end = time.time()
      print(f"Completed in {end - start:.2f} seconds")
      print(f"CPU results: {cpu_results}")
      print(f"IO results: {io_results}")

  # Run the example
  if __name__ == "__main__":
      asyncio.run(main())
  ```

* **Caching Strategies**:
  * Implement effective caching for expensive operations:

  ```python
  import functools
  import time
  from typing import Any, Callable, Dict, Hashable, Optional, TypeVar, cast
  
  T = TypeVar("T")
  
  # Simple in-memory cache with TTL
  class Cache:
      def __init__(self):
          self._cache: Dict[Hashable, tuple[Any, float]] = {}
          
      def get(self, key: Hashable) -> Optional[Any]:
          """Get a value from cache if it exists and is not expired."""
          if key not in self._cache:
              return None
          
          value, expiry = self._cache[key]
          if expiry and time.time() > expiry:
              # Value has expired
              del self._cache[key]
              return None
              
          return value
          
      def set(self, key: Hashable, value: Any, ttl: Optional[float] = None) -> None:
          """Set a value in the cache with optional TTL in seconds."""
          expiry = time.time() + ttl if ttl else None
          self._cache[key] = (value, expiry)
          
      def clear(self) -> None:
          """Clear all cached values."""
          self._cache.clear()
  
  # Global cache instance
  _global_cache = Cache()
  
  # Decorator for caching function results
  def cached(ttl: Optional[float] = None) -> Callable:
      """
      Cache function results with an optional TTL.
      
      Args:
          ttl: Optional time-to-live in seconds
          
      Returns:
          Decorator function
      """
      def decorator(func: Callable[..., T]) -> Callable[..., T]:
          @functools.wraps(func)
          def wrapper(*args: Any, **kwargs: Any) -> T:
              # Create a hashable key from function name and arguments
              key = (func.__name__, args, frozenset(kwargs.items()))
              
              # Check cache first
              cached_result = _global_cache.get(key)
              if cached_result is not None:
                  return cast(T, cached_result)
                  
              # If not in cache, execute function
              result = func(*args, **kwargs)
              
              # Store in cache
              _global_cache.set(key, result, ttl)
              
              return result
          return wrapper
      return decorator
  
  # Usage example
  @cached(ttl=60)  # Cache results for 60 seconds
  def expensive_operation(x: int, y: int) -> int:
      """An expensive operation to be cached."""
      print(f"Executing expensive operation with {x}, {y}")
      time.sleep(2)  # Simulate expensive computation
      return x ** y
  
  # Test the caching
  def test_caching():
      # First call is not cached
      start = time.time()
      result1 = expensive_operation(2, 8)
      duration1 = time.time() - start
      
      # Second call should be cached
      start = time.time()
      result2 = expensive_operation(2, 8)
      duration2 = time.time() - start
      
      # Different arguments should not be cached
      start = time.time()
      result3 = expensive_operation(3, 7)
      duration3 = time.time() - start
      
      print(f"First call: {result1}, took {duration1:.2f}s")
      print(f"Second call: {result2}, took {duration2:.2f}s")
      print(f"Different args: {result3}, took {duration3:.2f}s")
  
  # Run the test
  if __name__ == "__main__":
      test_caching()
  ```

* **Memory Optimization**:
  * Reduce memory usage with generator-based processing:

  ```python
  import csv
  import gzip
  from typing import Dict, Generator, Iterable, List, TypeVar

  T = TypeVar('T')

  def process_large_file(
      file_path: str,
      batch_size: int = 10000
  ) -> Generator[List[Dict[str, str]], None, None]:
      """
      Process a large CSV file in batches using generators.
      
      Args:
          file_path: Path to the CSV file (can be gzipped)
          batch_size: Number of rows to yield at a time
          
      Yields:
          Batches of dictionaries representing CSV rows
      """
      # Detect if file is gzipped
      open_func = gzip.open if file_path.endswith('.gz') else open
      
      with open_func(file_path, 'rt', newline='') as f:
          reader = csv.DictReader(f)
          batch = []
          
          for row in reader:
              batch.append(row)
              
              if len(batch) >= batch_size:
                  yield batch
                  batch = []
                  
          # Yield remaining items
          if batch:
              yield batch

  def chunked_processing(items: Iterable[T], chunk_size: int = 1000) -> Generator[List[T], None, None]:
      """
      Process any iterable in chunks to reduce memory usage.
      
      Args:
          items: Input iterable to process
          chunk_size: Size of each chunk
          
      Yields:
          Chunks of the input iterable
      """
      chunk = []
      
      for item in items:
          chunk.append(item)
          
          if len(chunk) >= chunk_size:
              yield chunk
              chunk = []
              
      # Yield remaining items
      if chunk:
          yield chunk

  # Example usage for processing a large dataset
  def process_user_data(file_path: str):
      """Process a large user dataset with optimized memory usage."""
      total_processed = 0
      
      # Process in batches to avoid loading the entire file
      for batch in process_large_file(file_path, batch_size=5000):
          # Further chunk the processing to control memory usage
          for users_chunk in chunked_processing(batch, chunk_size=100):
              # Process each small chunk
              result = process_user_chunk(users_chunk)
              
              # Update tracking
              total_processed += len(users_chunk)
              
          # Periodic status report without storing all data
          print(f"Processed {total_processed} records so far...")
      
      print(f"Completed processing {total_processed} total records")

  def process_user_chunk(users: List[Dict[str, str]]) -> List[Dict]:
      """Process a small chunk of users (implementation details)."""
      # Actual processing logic here
      return [process_user(user) for user in users]

  def process_user(user: Dict[str, str]) -> Dict:
      """Process a single user record."""
      # Example transformation
      return {
          'id': int(user.get('id', 0)),
          'name': user.get('name', '').strip().title(),
          'active': user.get('status', '').lower() == 'active',
          # Add more transformations as needed
      }
  ```

---

## See Also
- [Python Core Principles](/python/python-core-principles.md)
- [Python Code Quality](/python/python-code-quality.md)
