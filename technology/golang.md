# golang notebook

waitGoroup 其实是一种多线程等待机制。每个任务异步时 add(1)。各自任务完成时调用 Done()， 任务执行后 wait()；

所有的子任务全结束后，才会执行下一步。
