```javascript

// Declare global variables

int readers_waiting = 0

int writers_waiting = 0

int readers_active = 0

int writers_active = 0

mutex readers_mutex

mutex writers_mutex

mutex read_count_mutex

// Reader function

reader() {

  // Wait until there are no writers waiting

  acquire(writers_mutex)

  release(writers_mutex)

  // Increment the number of waiting readers

  acquire(readers_mutex)

  readers_waiting++

  release(readers_mutex)

  // Wait until there are no active writers

  acquire(read_count_mutex)

  while (writers_active > 0) {

    wait(read_count_mutex)

  }

  // Increment the number of active readers and decrement waiting readers

  acquire(readers_mutex)

  readers_active++

  readers_waiting--

  release(readers_mutex)

  // Read data

  // Decrement the number of active readers

  acquire(readers_mutex)

  readers_active--

  release(readers_mutex)

  // If there are no more active readers, signal waiting writers

  if (readers_active == 0) {

    signal(write_count_mutex)

  }

}

// Writer function

writer() {

  // Wait until there are no active readers or writers

  acquire(writers_mutex)

  while (writers_active > 0 || readers_active > 0) {

    writers_waiting++

    wait(writers_mutex)

    writers_waiting--

  }

  // Increment the number of active writers

  writers_active++

  // Write data

  // Decrement the number of active writers

  writers_active--

  // Signal waiting readers and writers

  if (readers_waiting > 0) {

    signal(read_count_mutex)

  } else {

    signal(write_count_mutex)

  }

}

```


  
 


### 
In this approach, readers and writers use separate mutexes to control access to the shared resource. When a reader arrives, it first checks to see if there are any active writers. If not, it increments the number of waiting readers and waits until there are no active writers. Once there are no active writers, it increments the number of active readers and reads the data. When finished, it decrements the number of active readers and signals any waiting writers if there are no more active readers.

###
When a writer arrives, it first waits until there are no active readers or writers. Once there are no active readers or writers, it increments the number of active writers and writes the data. When finished, it decrements the number of active writers and signals any waiting readers or writers.
